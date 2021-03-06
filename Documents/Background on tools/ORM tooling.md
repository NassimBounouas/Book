# Background on tools : **How are modern ORMs developed nowadays, in term of Organization, Architecture and Design ?**

# Questions

* How do components evolve over time \(including ownership, bugs, build stability, metrics\) ?

* Are dialects and caching highly coupled with other components  ?

* How do Hibernate and Entity Framework deal with dialects? Are there any major differences?

* How do Entity Framework and Hibernate manage caching? Are there any major differences?

# Methodology

we'll adopt a top-down methodology on the usage of our toolings, since the toolings are usable on the two frameworks we're gonna  use them on hibernate as an example, first we'll see the communications between the developers on the whole project, then we'll see on the hibernate-core module, and precisely on the dialect package, then we'll dive into the dialect to see how the classes evolving and which informations can we retreive from such analyses.

after we'll stay at the same level \(i.e class level\), in order to see the dialect classes impacted by pull requests, and the commits into those classes that are breaking the build.

finally we'll expose how we'll generate UML diagram and then extract relevent parts

# Tooling

## code maat / codescene.io

we started using codescene which gave us the first insight about the lead developer about the whole framework as this diagram shows us the interactions between the developers :

![](/assets/lead.png)

we can clearly identify that Steve Ebersol is the lead of the hibernate project, but let's see if its still the same from a feature point of view, for example the dialect feature is mainly developed through the hibernate-core module, and precisely through its dialect package, here's a visualization of the knowledge and code owner :

![](/assets/ownership.png)

this confirms what we saw on a higher-level through communications between developers, it seems a bit normal, since handling dialects is a core feature hence the delegation to the most experienced developer\(s\).

now let's have a look to logical coupling inside the dialect package, with codescene.io \(which uses code Maat under the hood, and some extra features\) we can have nice view about the coupling  :

![](/assets/logicalcoupling-dialect.png)

this is part of the logical-coupling digram \(we used here a part, since the original one is quiet huge, and will only be used on the book\).

As we can see, the Oracledb 9 Dialect is coupled to the Oracledb 8 Dialect \(since its an update, it must inherit most of its behaviour\), the SchemaUpdate and SchemaValidator are coupled too, but the interesting information is that Schema validation/update is not coupled to the implementation of the Dialects \(which is, by the way, a good news.\).

As we know, the software industry is always evolving, for example, each 2 or 3 months we hear of new version realese for postgresql, mysql, mariadb etc..., so handling the dialects appropriatly is a vital matter, since each version may introduce new elements to the dialects.

For example let's see how Hibernate deals with the postgresql releases that impacts the dialects, two well known versions that had impact are the 8.1 and the 9.3, first let's see how the complexity of the postgresql 8.1 evoles over the time :

![](/assets/postgresql-complexity.png)

this shows us, that the postgresql 8.1 is not that active over time \(in term of LoC\), but these additions introduced a lot of complexity \(majority of it are fixes since the 8.1 is EOL since 2010\), now let's see how the 9.3 behaves in comparaison of the 8.1 :

## [http://jenkinsapi.readthedocs.io/en/latest/build.html?highlight=commit](http://jenkinsapi.readthedocs.io/en/latest/build.html?highlight=commit)![](/assets/comparatif-postgresql.png)

the PostgreSQL93Dialect.java is so small that code maat can't chart it, this shows us that the 9.3 inherits all its behavior from the elder versions, and this without opening any java class.

## Github API :

the Github API will provide us with all the pull requests made on the projects, for example to access the pull requests resource on the hibernate project we'll follow the link : [https://api.github.com/repos/hibernate/hibernate-orm/pulls](https://api.github.com/repos/hibernate/hibernate-orm/pulls)

the most interesting part for us is the property "merge\_\_commit\_sha" which contains the sha1 code of the commit, with a groovy or python script we can iterate over the json object in order to retrieve the sha1 code then execute on the script the following command :

```
git show <sha1_code>
```

which will list all the modified files on the pull request.

this tooling seems quite appropriate in order to see if the selected features are subject to a lot of pull requests, and in case of the identified pull requests, see their type : bugfix, enhancement etc...

## Jenkins :

jenkins is a CI tool, but it can be used to track back the commits that broke the builds, and automate it with a python script using the Jenkins API : [http://jenkinsapi.readthedocs.io/en/latest/build.html?highlight=commit](#httpjenkinsapireadthedocsioenlatestbuildhtmlhighlightcommit-)

when obtaining the commit id, related to our features we can track back the files responsible of breaking the build, this will give us an idea of the stability of those features over the time \(since the builds are running with frequency\).

## Intellij IDEA visual paradigm plugin :

the idea behind using intellij with visual paradigm is to auto generate UML diagram from raw java code, then after converting it into XML format parse it to extract the relevant modules.

we tried our solution into a part of hibernate \(hibernate-core, this is what gave us insights about the dialect package and classes\), it seems quite convinient, but on the other hand the risky part is the transformation writing, since it can be a bit tricky.

# Issues

issues and problems we faced mainly in finding the right APIs \(i.e github api on pull requests\) and the design of the scripts, and on  how should we extract classes from UML digram.

these issues are not major as they will disapear once we'll get the python and groovy script to a stable version. on the other hand, extraction of the relevent parts of UML diagrams seems to be possible through different way, we still need decide which way to do it \(XML parsing vs manual treatment\).

