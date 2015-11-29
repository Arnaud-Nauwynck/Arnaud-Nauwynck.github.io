---
layout: post
title:  "Live Coding - Java Coding Exercise: a tool for synchronizing 2 dirs (backup) using SHA comparisons"
date:   2015-11-21 17:00:00
categories: 
tags: live-coding exercise video backup
---



Link of the Live Coding video lasting 28 minutes  
 <A href="{{site.url}}/assets/posts/2015-11-21-coding-exercise/live-coding.mkv">live-coding.mkv</A>  



<h1>Introduction : Live Coding - Student Exercise Correction</h1>

I asked my students to code in Java a "File Synchronisation Tool", using SHA-1 comparisons.

Basically, after the course on java.util.List, java.util.HashMap and java.io.* , I wanted to give them an exercise to practise what they had learned.<BR/> 

Students level in java are varying from beginners to intermediates. 
They are 20-21 years old, and working part-time in a company, and part time at university (3 years after french graduate "baccalaureat")

They had approximatly 1 month to give me their work, as an eclipse source code project, including tests.


<h1>Description of the Synchronizing Tool</h1>

The "File Synchronisation Tool" must compare recursively 2 directories (aka sourceDir and destinationDir), and compute the list of copy/removal files commands to get the destination directory be in sync with the source directory.

For example, suppose you want to compare source directory "/data/dir1" to destination directory "/data/dir1.backup", 
and you have some files added in source "/data/dir1/a/foo-new.txt", modified "/data/dir1/a/bar-modified.txt", and deleted "/data/dir1/a/baz-deleted.txt" (does not exists in source dir, only in "/data/dir1.backup/a/baz-deleted.txt")   

The tool ouput should be a list of commands to synchronize from source to dir:
{% highlight text %}
cp '/data/dir1/a/foo-new.txt'       '/data/dir1.backup/a/foo-new.txt'
cp '/data/dir1/a/bar-modified.txt'  '/data/dir1.backup/a/bar-modified.txt'
rm                                  '/data/dir1.backup/a/baz-deleted.txt'
{% endhighlight text %}



<BR/>
The synchronisation tool must work in a "3-way" algorithm style, by first computing all SHA-1 signatures of source files, then SHA-1 signatures of destination files... then find matches and mismatches between source and destinations.
It could(should) also warn on duplicate file contents found in source, so possibly use symlink on destination instead of wasting disk copy.

It was explicitly stated that the internal datastructure to use was a Map of List, whose key is the SHA-1 of file content, and values are the related paths of all files having the SHA.

{% highlight java %}
Map<String/*SHA*/, List<String/*Path*/>> sha2RelativePaths = 
   new HashMap<String, List<String>>();   
{% endhighlight java %}
 

As as example, suppose you have a directory with the following content
{% highlight text %}
/
/a/foo.txt             with SHA1= a1b2c...  512 hexa chars code
/a/foo-copy.txt                 = (idem)
/a/b/bar.txt                SHA1= e7f8a...
{% endhighlight text %}

then the data signature of theses files would be a Map with the following content:
{% highlight text %}

{
   { key1: "a1b2c...",  value: List[ "/a/foo.txt",  "/a/foo-copy.txt"  ] }
   { key2: "e7f8a...",  value: List[ "/a/b/bar.txt" ]
}
{% endhighlight text %}
 

<h1>Code Difficulty and Coding Speed Performance</h1>

The code is pretty simple, as the following shell command on the correction source code shows it:

{% highlight text %}
$ wc -l ./src/fr/iut/tp/backupsha/BackupSHAMain.java ./src/fr/iut/tp/backupsha/BackupSHAMainTest.java 
 134 ./src/fr/iut/tp/backupsha/BackupSHAMain.java
  54 ./src/fr/iut/tp/backupsha/BackupSHAMainTest.java
 188 total
{% endhighlight text %}


The JUnit tests is here:
{% highlight java %}
public class BackupToolMainTest {

    // sut= System Under Test
    BackupToolMain sut = new BackupToolMain();
    
    private static final String SHA1_FOO = "0beec7b5ea3f0fdbc95d0dd47f3c5bc275da8a33";
    private static final String BAR_SHA = "62cdb7020ff920e5aa642c3d4066950dd1f01f4d";
    
    @Test
    public void testComputeSHA() throws Exception {
        // Prepare
        // Perform
        String res = sut.computeSHAFile(new File("test/dir1/foo.txt"));
        // Post-check
        Assert.assertEquals(SHA1_FOO, res);
    }

    @Test
    public void testComputeSHA_fooCopy() throws Exception {
        // Prepare
        // Perform
        String res = sut.computeSHAFile(new File("test/dir1/subdir/foo-copy.txt"));
        // Post-check
        Assert.assertEquals(SHA1_FOO, res);
    }

    @Test
    public void testRecursiveSHAToPathes() {
        // Prepare
        // Perform
        Map<String, List<String>> res = sut.recursiveSHAToPathes(new File("test/dir1"));
        // Post-check
        Assert.assertEquals(2,  res.size());
        List<String> lsFoos = res.get(SHA1_FOO);
        Assert.assertEquals(2, lsFoos.size());
        Assert.assertEquals("/subdir/foo-copy.txt", lsFoos.get(0));
        Assert.assertEquals("/foo.txt", lsFoos.get(1));
        
        List<String> lsBar = res.get(BAR_SHA);
        Assert.assertEquals(1, lsBar.size());
        Assert.assertEquals("/bar.txt", lsBar.get(0));
    }
    
    @Test
    public void testCompare() {
        // Prepare
        sut.setSrcDir("test/dir1");
        sut.setDestDir("test/dir2");
        // Perform
        List<String> res = sut.compare();
        // Post-check
        Assert.assertEquals(4, res.size());
        Assert.assertEquals("rm   'test/dir2/subdir/foo-copy.txt'", res.get(3));
        Assert.assertEquals("rm   'test/dir2/subdir/file-added.txt'", res.get(2));
        Assert.assertEquals("cp 'test/dir1/bar.txt'  'test/dir2/bar.txt'", res.get(0));
        Assert.assertEquals("cp 'test/dir1/subdir/foo-copy.txt'  'test/dir2/subdir/foo-copy.txt'", res.get(1));
    }
}
{% endhighlight java %}


And the Java code outline is here

Basically, there is a main() to scan the 2 directories and compare the 2 Maps by SHA-1 of List of paths.

{% highlight java %}
public class BackupToolMain {

    private String srcDir = "test/dir1";
    private String destDir = "test/dir2";
    
    public static void main(String[] args) {
        BackupToolMain app = new BackupToolMain();
        app.parseArgs(args);
        app.run();
    }

    private void parseArgs(String[] args) {
        for (int i = 0; i < args.length; i++) {
            String arg = args[i];
            if (arg.equals("-i")) {
                srcDir = args[++i];
            } else if (arg.equals("-o")) {
                destDir = args[++i];
            } else {
                throw new RuntimeException("unrecognised arg");
            }
            
        }
    }
    private void run() {
        List<String> syncCommands = compare();
        System.out.println(syncCommands);
    }

    /*pp*/ List<String> compare() {
        Map<String,List<String>> sha2pathsSrc = recursiveSHAToPathes(new File(srcDir));
        Map<String,List<String>> sha2pathsDest = recursiveSHAToPathes(new File(destDir));
        List<String> syncCommands = compare(sha2pathsSrc, sha2pathsDest);
        return syncCommands;
    }

    private List<String> compare(Map<String, List<String>> sha2pathsSrc, Map<String, List<String>> sha2pathsDest) {
        List<String> resSyncCommands = new ArrayList<String>();
        // scan 1: from destDir, find missing elt in srcDir
        for(Map.Entry<String,List<String>> eDest : sha2pathsDest.entrySet()) {
            String sha = eDest.getKey();
            List<String> destPathes = eDest.getValue();
            List<String> srcPathes = sha2pathsSrc.get(sha);
            if (srcPathes == null) {
                srcPathes = Collections.emptyList();
                compareListPathes(sha, srcPathes, destPathes, resSyncCommands);
            }
        }
        // scan 2: from srcDir .. find corresponding elt in destDir
        for(Map.Entry<String,List<String>> eSrc : sha2pathsSrc.entrySet()) {
            String sha = eSrc.getKey();
            List<String> srcPathes = eSrc.getValue();
            List<String> destPathes = sha2pathsDest.get(sha);
            if (destPathes == null) {
                destPathes = Collections.emptyList();
            }
            compareListPathes(sha, srcPathes, destPathes, resSyncCommands);
        }
        return resSyncCommands;
    }

    private void compareListPathes(String sha, List<String> srcPathes, List<String> destPathes, List<String> resSyncCommands) {
        // scan 1: from destDir, find missing elt in srcDir
        for(String path : destPathes) {
            if (! srcPathes.contains(path)) {
                resSyncCommands.add("rm   '" + destDir + path + "'");
            }
        }
        // scan 2: from srcDir .. find corresponding elt in destDir
        for(String path: srcPathes) {
            if (! destPathes.contains(path)) {
                resSyncCommands.add("cp '" + srcDir + path + "'  '" + destDir + path + "'");
            }
        }
    }

    /*pp*/ Map<String, List<String>> recursiveSHAToPathes(File file) {
        Map<String, List<String>> res = new HashMap<String, List<String>>();
        recursiveSHAToPathes(file, "", res);
        return res;
    }

    private void recursiveSHAToPathes(File currDir, String currPath, Map<String, List<String>> res) {
        for(File f : currDir.listFiles()) {
            String childPath = currPath + "/" + f.getName();
            if (f.isDirectory()) {
                // recurse
                recursiveSHAToPathes(f, childPath , res);
            } else if (f.isFile() && f.canRead()) {
                try {
                    String sha = computeSHAFile(f);
                    List<String> ls = res.get(sha);
                    if (ls == null) {
                        ls = new ArrayList<String>(1);
                        res.put(sha, ls);
                    }
                    ls.add(childPath);
                } catch(Exception ex) {
                    System.err.println("Failed to read file '" + f + "' ...ignore, no rethrow");
                }
            } else {
                System.err.println("Can not read file '" + f + "' ...ignore");
            }
        }
    }

    /*pp*/ String computeSHAFile(File file) throws Exception {
        MessageDigest md = MessageDigest.getInstance("SHA-1");
        try(InputStream in = new FileInputStream(file)) {
            byte[] buffer = new byte[4096];
            int len;
            while((len = in.read(buffer)) != -1) {
                md.update(buffer, 0, len);
            }
            byte[] bytes = md.digest();
            return DatatypeConverter.printHexBinary(bytes).toLowerCase();
        } 
    }
{% endhighlight java %}




The main part takes 151 lines of code, and the JUnit tests 73, which makes a total of 224 lines.

You might wonder that it is not so much, and it should take ~ 30 seconds per line to code, so ~1 hour to get it...<BR/> 
Indeed, it took me ~2 hours the first time, 1 hour the second time... and 35mn the 5th time.<BR/>
Being fast is possible only because you repeat something you already knows, or knows how to do it... It is just the repetition and training that makes you becoming fast, like every athletes.
<BR/>

Notice that the 3-th and 4-th times, it actually took me more times than before (~1h30mn) because I added more JUnit tests, checked with linux bash command "sha1sum", and I found a bug in my hexa decimal conversion from byte[] to String... then I learned a simpler way and safer to code it. 
<BR/>
The 5th times, I did a small mistake in my code (forgot to scan for deletions before additions/modifications), so the result was almost good, but incorrect if you try to execute it for real. 
<BR/>
The video provided is the 5-th one.


If my students were also experimented developper, it should take them also few hours... but for them, it was their first coding of such tool ... I received by email the first projects (by asking several times) after more than 15 days, but I think they have spend only few (~3 ?) days on it.<BR/>
Unfortunatly, only 4 out of 18 students had done something, and the results submitted were FAR FAR from expected, both in term of results, performance optimisations and code readability/quality !!<BR/>

I had to reply to them several times, to explain what was wrong in their programs.<BR/>
Basically, the forgot to do a recursive scan of the directory, they only listed 1 level of directory. Then they failed to understand that the SHA1 was computed on the file content, that it should be converted to hexa string (or whatever array class in java implementing equals() and hashCode() ), and that they had to handle the possibility of file content duplicated several times (they used Map<String,String> instead of Map<String,List<String>>) ...




<h1>More on Live Coding Video Recording ... </h1>

It took me hours to compress my video file into a usable, standard, and not "too big" file !!!<BR/>
I have started with a recorded file in a proprietary format (non standard, not usable for web users..) which was 30Mo.<BR/>
I could only convert from this format to ".MOV" file which was 1.5 Go (not usable for web users, for other reasons)<BR/>
I ended it up after several days in a MP4 file of ~ 22Mo ... but I was still very disapointed by the result<BR/>

Obviously ".MOV" format was not adapted... and/or the compression was not done with the correct parameters. Clearly, MOV is adapted to  movies (successions of natural photos), and JPEG compression is pretty good at that. But for artificial computer graphics images, it is a pity... and a shame, because such images are extremely simple (only lines, and text font), so they "should be" compressed much much more! Not only the spatial redundncy is high, but also the temporarl one (things changes very slowly on the screens, and only in the regions near the mouse / keyboard input focus). 
<BR/>

And because I am curious about everything in computer sciences, I have spent the last few past-days/current-weeks(/incoming-monthes) searching for tools, documentations, research papers on the subject of "screencast video compression" !

I am convinced there is a better solution...


More on this in following post: <A href="{{site.url}}/2015/11/29/screencast-recording-compression.html">Screencast Recording & Dedicated Computer Graphics Video Compression</A>





