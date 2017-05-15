---
layout: post
title:  "Using Files from web browser and AngularJS: Select, Drag'n Drop, Read content, Upload to springboot"
date:   2017-05-16 08:20:00
categories: 
tags: web AngularJs file springboot
---


<H1>Files sandbox possibilities in Web Browser</H1>

Due to security restrictions, it is not possible from a web browser to perform Javascript operations on files, 
unless the user has manually choosen a file.

This little demo project shows how to perform operations on files using AngularJS (and springboot on server-side):
<ul>
<li>Select files using html "&lt;input type="file /&gt;"</li>
<li>Drag'n Drop files</li>
<li>Read file content</li>
<li>Upload file content to server</li>
</ul>

The full repository code is available here:
<A href="https://github.com/Arnaud-Nauwynck/test-snippets/tree/master/test-web-js-files">https://github.com/Arnaud-Nauwynck/test-snippets/tree/master/test-web-js-files</A>

Here is the Angular screen to perform tests:
<img src="{{site.url}}/assets/posts/2017-05-16-test-web-js-files/screenshot-test-web-js-files.png"/>



<H1>Select files using html "&lt;input type="file" /&gt;"</H1>

{%highlight html %}
    <input type="file" multiple
    	set-model-on-change ng-model="ctrl.selectedFiles1"  
    	ng-change="ctrl.onInputFileChange()">
    
    <!-- using pure js event binding to angular.. 
    <input type="file" multiple
    	onchange="angular.element(this).scope().ctrl.addSelectedFiles2(this.files)">
    -->
    
    <h2>Files</h2>
    <div ng-repeat="file in ctrl.selectedFiles1">
      {{file.name}}
    </div>
{%endhighlight html %}

Usually, ng-model binding works... except for this input type (??)</BR>
You simply use the javascript File[] from your controller. It contains "trusted javascript file", than you can read. 

{%highlight javascript %}
.controller('MyController', function($scope, $http) {
    var self = this;
    self.selectedFiles1 = []; // from input
    
    self.onInputFileChange = function() {
      console.log("onInputFileChange: selected files: ", self.selectedFiles1);
    };
{%endhighlight javascript %}


The directive "set-model-on-change" is custom, to bind "change" DOM events to angular for the input type="file".
{%highlight javascript %}
.directive("setModelOnChange", function() {
  return {
    require: "ngModel",
    link: function postLink(scope,elem,attrs,ngModel) {
      elem.on("change", function(e) {
        console.log("on change (from directive)", e);  
        var files = elem[0].files;
        ngModel.$setViewValue(files);
      })
    }
  }
})
{%endhighlight javascript %}

You can see log in chrome dev tools after selecting 2 files:
<img src="{{site.url}}/assets/posts/2017-05-16-test-web-js-files/screenshot-chrome-log-selected-files.png"/>


    
<H1>Drag'n Drop files</H1>

Another nice way of selecting Files is to use Drag'n Drop: you drag a file from your system file explorer, 
and drop it into a html Drop zone that accept you files.<BR/>

Here is a drop-zone, using a custom AngularJS directive.<BR/>
This directed is adapted from draganddrop.js, <em>/*! Angular draganddrop v0.2.2 | (c) 2013 Greg Bergé | License MIT */</em><BR/>
I have adapted it because I could not make it work on the filtering of accepted mime-types files. 

{%highlight html %}
	<div id="drop_zone" style='border: solid'
 		drag-over="ctrl.onDragOver($event)" drag-over-class="drag-over"
 		drop-accept="ctrl.dropAccept($event)"
		drop="ctrl.onDrop($data, $event)" drop-effect="link" 
		>Drop files here...</div>

{%endhighlight html %}

In the js-part, I implements the 3 callbacks method onDragOver(), dropAccept() and onDrop().<BR/>
The important one is onDrop() that simply get the dropped files and store them as angularJs model values (avoiding duplicates files based on their name+size+modifDate).
Notice that the file full path can not be consulted from Javascript, only the name.

{%highlight javascript %}
	  self.selectedFiles2 = []; // trusted javascript File are appended when dropping files, from self.onDrop()
	  
	  // optionnal, to add logs
      self.onDragOver = function($event) {
          // var files = $event.dataTransfer.files;
          // console.log("onDragOver (using draganddrop.js module directive)", {event: $event, files: files} );  
      };
      // optionnal, to decide if content files can be dropped. Should filter on mime-types.
      self.dropAccept = function($event) {
          // console.log("dropAccept (using draganddrop.js module directive)", $event);
          // var files = $event.dataTransfer.files;
          return true;
      }
      self.onDrop = function(data, $event) {
          var files = $event.dataTransfer.files;
          console.log("onDrop (using draganddrop.js module directive)", {data, $event, files});
          // self.selectedFiles2.push(...files); do not add doublon..
          for(var i = 0; i < files.length; i++) {
              var f = files[i];
              if (-1 === findFileElt(self.selectedFiles2, f)) { // indexOf() does not work!
                  self.selectedFiles2.push(f);
              } else {
                  console.log('file already added ', f);
              }
          }
      };
      
      function findFileElt(ls, elt) {
          for(var i = 0; i < ls.length; i++) {
             if (ls[i].name === elt.name && // can not compare by full path?!!
             ls[i].size === elt.size && ls[i].lastModified === elt.lastModified) {
                 return i;
             }   
          }
          return -1;
      }
{%endhighlight javascript %}

You can see logs in chrome:<BR/>
<img src="{{site.url}}/assets/posts/2017-05-16-test-web-js-files/screenshot-chrome-log-dropped-files.png"/>


<H1>Read Files content</H1>

After selecting/dropping files, click on button "Read selected files", and the content will be read and displayed in textarea.

<img src="{{site.url}}/assets/posts/2017-05-16-test-web-js-files/screenshot-read-files-content.png"/>

Here is the code in html
{%highlight html %}
    <button ng-click="ctrl.onReadSelectedFiles()">Read selected files</button>
    
    <div ng-repeat="fc in ctrl.selectedFileWithContents">
      {% raw %}{{fc.name}}{% endraw %}:
	  <textarea cols='30' rows='5'>{% raw %}{{fc.textContent}}{% endraw %}</textarea>
    </div>
{%endhighlight html %}

In Javascript, it is a bit more difficult because all I/O reading are asynchronous methods, with callbacks.<BR/>
Also notice that the callback need to call angularJS "$scope.apply()" to perform a refresh. 

{%highlight javascript %}
    self.onReadSelectedFiles = function() {
      self.selectedFileWithContents = [];
      var files = [...self.selectedFiles1, ...self.selectedFiles2];
      for(var i = 0; i < files.length; i++) {
          var file =  files[i];
          console.log("async read file[" + i + "]", file);
          
          var reader = new FileReader();
          reader.onload = (function(f) {
            return function(e) {
                $scope.$apply(function() { // ugly.. force angular refresh!! 
                  var textContent = e.target.result;
                  console.log("finished read file[" + i + "]",{ name: f.name, textContent: textContent});
                  self.selectedFileWithContents.push({ name: f.name, textContent: textContent});
                });
            };
          })(file);
          reader.readAsText(file);
          
      }
    };

{%endhighlight javascript %}

If you need to load multiple files, and call a single callback at the end, you have to chain callbacks... <BR/>
A common way for that is to use $q Promises, and resolve "$q.all([ promise0, promise1 ]).then(..)".    


<H1>Upload file content to server</H1>

There are many ways to submit content data to server.<BR/>
This can be a multi-part files, or simply a http Rest body request with "Content-type: application/*whatever*" ...but maybe not "application/json" which is default when using AngularJS.

You can event wrap the bytes array content into one a the field of a regular json object, and send the json..

<H2>Uploading using MultipartFile</H2>

{%highlight javascript %}
  self.onUploadFilesMultipartFiles = function() {
    var files = [ ...self.selectedFiles1, ...self.selectedFiles2 ];
    var formData = new FormData();
    for(var i = 0; i < files.length; i++) {
      formData.append(files[i].name, files[i]);
    }
    $http.post('/app/uploadMultipartFiles', formData, {
      transformRequest: angular.identity,
      headers: { 'Content-Type': undefined }
    }).then(function (data) {
      self.uploadStatus += 'OK uploaded ' + files + ' using MultipartFile\n';            
    }, function (err) {
      self.uploadStatus += 'Failed uploaded ' + files + ' using MultipartFile' + err + '\n';            
    });
  };
{%endhighlight javascript %}

In springboot, 
{%highlight java %}
    @PostMapping(value="/uploadMultipartFiles", consumes = {"multipart/form-data"})
	public void uploadMultipartFile(MultipartHttpServletRequest request) throws IOException {
		LOG.info("/uploadMultipartFiles ");
		for(Map.Entry<String, MultipartFile> e : request.getFileMap().entrySet()) {
			String fileName = e.getKey();
			byte[] content = e.getValue().getBytes();
			logContentIfText(fileName, content);
		}
	}
{%endhighlight java %}


If you have to upload only a fixed number of file(1,2..) whith given part name (like "attachment1", "attachement3, "video"...), you can do this in springboot:
{%highlight java %}
    @PostMapping(value="/uploadMultipartFile", consumes = {"multipart/form-data"})
	public void uploadMultipartFile12(@RequestPart("file") MultipartFile file) throws IOException {
		LOG.info("/uploadMultipartFile " + file.getName() + " using MultipartFile");
		logContentIfText(file.getName(), file.getBytes());
	}
{%endhighlight java %}


<H2>Uploading using Request Body Content</H2>

This method is more natural for a Rest endpoint, in particular when using curl:

{%highlight shell %}
curl -X POST --data-binary @myfile.txt http://localhost:8080/upload/myfile.txt

#Notice... curl -X POST -d @myfile.txt .... will work byt prune all newline characters!!! not what you want for a csv file for example!! 
{%endhighlight shell %}
 
In Javascript, the difficulties are 1/ to read the file with async callback, then 2/ obtain a "Blob" to convert to "Int8Array", 
 submit using $http.post() without transforming to json, and finally change default http header to remove 'Content-Type: application/json'. 

{%highlight javascript %}
  self.onUploadFileBytes = function() {
    var files = [ ...self.selectedFiles1, ...self.selectedFiles2 ];
    for (var i = 0; i < files.length; i++) {
      var file = files[i];
      var fileName = file.name;
      console.log("async read file[" + i + "]: '" + fileName + "'", file);
      var reader = new FileReader();
      reader.onload = (function(f) {
        return function(e) {
          var bytesContent = new Int8Array(e.target.result);
          self.asyncUploadFileBytes(f.name, bytesContent);                    
        };
      })(file);
      reader.readAsArrayBuffer(file);
    }
  };

  self.asyncUploadFileBytes = function(fileName, int8ArrayContent) {
    $http.post('/app/uploadFileBytes/' + fileName, int8ArrayContent, {
      transformRequest: [],
      headers: { 'Content-Type': undefined }
    }).then(function(res) {
      self.uploadStatus += 'OK uploaded ' + fileName + '\n';               
    }, function(err) {
      self.uploadStatus += 'Failed uploaded ' + fileName + ' : ' + err + '\n';
    });
  };
{%endhighlight javascript %}

From java, it is easy to handle... and there are also several possibilities in springboot:
you can bind method parameters as HttpServletRequest, and @RequestBody InputStream, or as @RequestBody byte[].

The simpler in springboot is using byte[]: 

{%highlight java %}
	@PostMapping(value="/uploadFileBytes/{fileName:.+}", consumes=MediaType.ALL_VALUE)
	public void uploadFileBytes(@PathVariable(name="fileName", required=true) String fileName, @RequestBody byte[] inputBody) {
		LOG.info("/uploadFile '" + fileName + "' using byte[]");
		logContentIfText(fileName, inputBody);
	}
{%endhighlight java %}

2 remarks in java: 
<ul>
<li>the @PathVariable by default "{fileName}" would prune the file name extension... you have to use a regular expression: "{fileName:.+}"</li>
<li>you need to override the default "application/json" to accept all mime-types:  consumes=MediaType.ALL_VALUE </li>
</ul>

 

<H1>Conclusion</H1>

Javascript File API works great, and are really simple !
<BR/>

SpringBoot and Angular(JS) are awesome.
<BR/>



