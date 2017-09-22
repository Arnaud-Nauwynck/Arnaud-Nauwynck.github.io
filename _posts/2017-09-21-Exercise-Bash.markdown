---
layout: post
title:  "Exercise on Bash"
date:   2017-09-20 08:00:00
categories: 
tags: shell bash scripts java
---

<H1>Exercise on Linux Shell Script (Bash)</H1>

<H2>Question 1</H2>
Préparation: récupérer le code source d'un porjet et dezipper le.

Par exemple apache-maven-3.5.0-src.zip depuis https://maven.apache.org/download.cgi

Ouvrer un terminal shell et aller dans le répertoire.


<H2>Question 2</H2>
- Lister récursivement avec la command "find" les fichiers et répertoires.
- Lister uniquement les fichiers (option "-f")
- Lister uniquement les fichiers avec le nom '\*.java'   (attention, '\*' est un caractere special du shell)
- Lister uniquement les fichiers avec le nom "\*.java", mais pas "\*Test.java"
- compter le nombre de fichers "\*.java" mais pas "\*Test.java"


Hints: utiliser les command "find . -f", "wc -l", "grep -v" 

<H2>Question 3</H2>
- Pour un fichier (java), afficher le nombre de ligne de code du fichier 
- Faire une boucle qui itere sur les fichiers et affiche pour chacun sa taille
- faire une boucle qui somme le nombre total de lignes des fichiers
- afficher seulement les 10 premieres lignes des des fichiers les plus gros avec leur taille   

- Facultatif: sans boucle mais avec la command "find . .. -exec .. \;", afficher la liste de tous les fichiers et leur taille

Hints:

{% highlight shell %}
for f in $( find . ); do echo $f; done
cat $f | wc -l
linecount=$( .. )
sum=$(( $sum + $linecount ))
sort -n
head -10
{% endhighlight %}

<H2>Question 4</H2>
Faire un programme java qui affiche la liste des arguments du main

{% highlight java %}
public class EchoMain {
  public static void main(String[] args) {
    for (int i = 0; i < args.length; i++) {
      System.ou.println("args[" + i + "]: '" + args[i] + "'");
    }
} 
{% endhighlight %}

L'executer depuis votre IDE (Eclipse, ..)

<H2>Question 5</H2>
Executer le programme en ligne de commande shell

{% highlight text %}
java -cp bin EchoMain
{% endhighlight %}

L'executer avec des arguments internes java (-D, -X, ...) et des arguments
par exemple: 
{% highlight text %}
java -cp bin -Xmx100m -Xms100m -Dprop1=value1 EchoMain Hello World -Dprop2=value2 
{% endhighlight %}


<H2>Question 6</H2>
Faire un script shell pour executer le programme java avec les paramètres passés au shell

file "javaecho.sh":
{% highlight text %}
#!/bin/bash
java -cp bin EchoMain $@
{% endhighlight %}


Le rendre executable ("chmod u+x javaecho.sh"), et le tester

- ./javaecho.sh Hello World

Les commandes suivantes fonctionnent-elles?

- javaecho.sh Hello World
- javaecho Hello World
- ./javaecho Hello World
- PATH="$PATH:."; javaecho.sh Hello World


<H2>Question 7</H2>

Tester (et expliquer) les résultats pour les paramètres suivant:

- ./javaecho.sh Hello World
- ./javaecho.sh Hello\ World
- ./javaecho.sh "Hello World"
- ./javaecho.sh "Hello\ World"
- ./javaecho.sh 'Hello World'
- ./javaecho.sh 'Hello\ World'
- ./javaecho.sh *
- ./javaecho.sh '*'
- ./javaecho.sh \*
- ./javaecho.sh '\*'
- i=1; ./javaecho.sh $1
- i=1; ./javaecho.sh "$1"
- i=1; ./javaecho.sh '$1'
- i=1; ./javaecho.sh \$1

<H2>Question 8</H2>

Tester la difference avec  @* au lieu de $@
 
script file "javaecho2.sh":
{% highlight text %}
#!/bin/bash
java -cp bin EchoMain $*
{% endhighlight %}


<H2>Question 9</H2>
Modifier le programme java pour rajouter a la fin  "Thread.sleep(10000); System.ou.println("exiting..);"

Pendant que le programme tourne (attend), vérifier que vous voyez le processus avec les command "ps", "ps aux", et  "ls /proc/<<pid>>"

<H2>Question 10</H2>

Faire un script shell "monitor-javaecho.sh" qui relance le programme quand celui-ci s'arrete

Hints: 

- utiliser "$!" pour connaitre le pid du dernier process démarré par le shell
- Faire un boucle toutes les secondes qui teste si le fichier "/proc/<pid>" exist


{% highlight text %}

./javaecho.sh @$
pid=$!
...
if [ ! -e /prod/$pid ]; then
	echo "le process est arreté.. relancer le"
fi 

{% endhighlight %}
