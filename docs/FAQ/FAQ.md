####1. *If I am already using gsDevKitHome, why should I consider moving to GsDevKit_home?*

GsDevKit_home has been made more modular with separate git projects for the tODE client and GemStone servers. With a new, modular directory structure, project-specific (private) files like session descriptions, tODE scripts and customized `system.conf` files are co-located in a separate git project [(GsDevKit_local_sys][5]) that can be managed in a private repository (i.e., [BitBucket][6]) keeping your team's private shared data private.
  Modules for development clients like [Jade][3] and [GemTools][4] can be added to GsDevKit_home, as well as modules for production tools like [Daemon Tools][8], [Monit][9], [Apache][10], [nginx][12], [lightppd][11], etc.

By default, git-based repositories used by GsDevKit_home are cloned thus reducing the reliance on network and making for more reliable builds. The ulitmate goal is to move all projects used by GsDevKit to [git][13] so that builds can be done without a network connection.

It is no longer necessary to fork the GsDevKit_home project as part of the installation process.

Better support for Windows users. The installation process for the Windows, Linux and OSX platforms is the same.  

[**COMMENTS**][28]

---
---
---
####2. *Do I have to rebuild all of my stones to use GsDevKit_home?*
No.
If you have a stone that was created with gsDevKitHome (either dev or master branches), you can use the `attachOldDevKitStone` script to put the old stone under GsDevKit_home management. 

For example the following command attaches to a stone named `dev_329` that was created using the dev branch of GsDevKitHome (`-d` option) where the old `$GS_HOME` is `/export/foos1/users/dhenrich/dev/dev_gsDevKitHome`:

```shell
attachOldDevKitStone -d -t dev_329 3.2.9 /export/foos1/users/dhenrich/dev/dev_gsDevKitHome
```

`attachOldDevKitStone` creates a symbolic link to the original stone directory and copies the original session description to the new `$GS_HOME/sys/local/sessions` directory. 

If the `-t` option is specied, the script:

 1. copies the tODE scripts and projects associated with the stone into the new location: `$GS_HOME/sys/stones/<stone-name>`. With the `-m` option, only the home directory is copied.
 2. makes `pre-rebuild` `backups` and `snapshots` of the stone and then rebuilds the in-image tODE structures to match the new GsDevKit_home directory structure. With the `-m` option no `snapshot` is made.
 3. mounts the old `$GS_HOME/tode` as `/oldTodeRoot` so that the old directory structure is accessible from within tODE.

As you can see in the output of the `status` command, the stone is tucked neatly into the new GsDevKit_home system:

```
foos:_home>status
Installed Products:
	3.1.0.6
	3.2.9
Installed Clients:
	tode	tODE client
	  todeClient.image
Available Sessions:
	dev_329
	gs_329
Installed Stones:
	3.2.9	dev_329
	3.2.9	gs_329
```

[**COMMENTS**][28]

---
---
---
####3. *Can I use a GsDevKit_home tODE client to connect to older stones?*
Yes. 
In general all tODE clients should be able to connect to older stones. 
Once you have run the `$GS_HOME/bin/installClient` or `$GS_HOME/bin/installServerClient` scripts, all you need to do is copy the session description into your `$GS_HOME/sys/local/sessions` directory.

For a *master branch* checkout of gsDevKitHome, the session descriptions are found in the old `$GS_HOME/tode/client/descriptions` directory.

For a *dev branch* checkout of gsDevKitHome, the session descriptions are found in the the old `$GS_HOME/tode/sys/local/client/descriptions` directory.

[**COMMENTS**][28]

---
---
---
####4. *When a new version of GsDevKit_home is published, how do I update my checkout?*
When a new version is published I send an [email announcement to the GLASS mailing list][30], where I will reference a specific pull request that was used to create the release like the following:

![GsDevKit_home pull request][31]

Each of my pull requests has an **Update Scripts** section where I will describe the script or scripts that should be run to update your GsDevKit_home checkout.
Typically the **Update Scripts** will involve running the `$GS_HOME/bin/updateGsDevScript` with a combination of the `-g`, `-t` and `-i` options (when in doubt run `$GS_HOME/bin/updateGsDevScript -g -t -i`).

#####updateGsDevScript options
When the `-g` option is specified, the clones for the GsDevKit_* modules (GsDevKit_home, GsDevKit_gs_client_dev, GsDevKit_todeClient, GsDevKit_gs_server, and GsDevKit_sys_local) that are currently being used are updated by doing a `git pull` from the appropriate *remote*.

When the `-t` option is specified, the clone of the tODE project is updated (`$GS_HOME/shared/repos/tode`).

When the `-i` option is specied the command line Pharo image (when `-g` option is specified) and/or the tode client images (when the `-t` option is specified) are rebuilt.

#####Following `Previous Pull Request` links
If you are curious about any pull requests that you might have missed, you can take note of the commit SHA in your current checkout using the `git log -1` command:

```
foos:GsDevKit_home>git log -1
commit d4fde700cddc2356815aac684ccc340e2384d6a7
Merge: 1876b7a 6c9c685
Author: Dale Henrichs <dale.henrichs@gemtalksystems.com>
Date:   Sat Oct 17 08:02:57 2015 -0700

    Merge pull request #3 from GsDevKit/dev
    
    todeUpdate customization ... another FAQ
```

and follow the **Previous Pull Request** link:

![GsDevKit_home pull request previous pull request link][33]

comparing the SHA of the *merge commit*:

![GsDevKit_home pull request merge detail][32]

until you find a match.


[**COMMENTS**][28]

---
---
---
####5. *When a new version of tODE is published, how do I update my checkout and my stones?*
To update your checkout, run the `$GS_HOME/bin/updateGsDevKit` with the `-t` option:

```Shell
updateGsDevKit -t
```

To update the version of tODE used in a stone, run the `$GS_HOME/bin/todeUpdate` command:

```Shell
todeUpdate <stone-name>
```

[**COMMENTS**][28]

---
---
---
####6. *Do I have to bootstrap GLASS1 and tODE from scratch every time I create a stone?*
No.

#####tODE stone
When you create a stone you can specify a path to a pre-built extent file, so if you want to avoid bootstrapping GLASS1 and tODE every time you create a stone you should first create a stone that you will use to manage your pre-built extent file. The following creates a tODE stone based on GemStone 3.2.9:

```
$GS_HOME/bin/createStone prebuilt_329 3.2.9
```

During the `createStone` for a tODE stone, a snapshot extent named `extent0.tode.dbf` is created.

This snapshot can be used as the starting point for another `createStone` thus bypassing the bootstrapping process:

```
$GS_HOME/bin/createStone -t $GS_HOME/server/stones/prebuilt_329/snapshots/extent0.tode.dbf new_329 3.2.9
```

When a new version of tODE is announced, you can update the prebuilt stone with the following:

```
$GS_HOME/bin/todeUpdate prebuilt_329
```

which also saves a new snapshot.

#####Non-tODE stone

If you want to prebuild an extent file that does not contain tODE, start by creating a stone using `$GEMSTONE/bin/extent0.seaside.dbf`:

```
$GS_HOME/bin/createStone -g prebuilt_329 3.2.9
```

and then load the rest of the code that you need bootstrapped.
You can then create the snapshot extent by following the instructions in [How to make an extent snapshot backup][30]. Thereafter you use the `-s` option and specify the path to the custom extent when you create a new stone:

```
$GS_HOME/bin/createStone -s <path-to-custom-snapshot> new_329 3.2.9
```

[**COMMENTS**][28]

---
---
---

####7. *How do I create a new project from scratch in tODE?*

For a project in tODE, you need a package for managing your source code (classes and extension methods), a *BaselineOf* for specifying the load order of packages and project dependencies, a git repository to manage versions of your source code on disk, and a project entry for specifying how you want tODE to treat your project.
The `project new` command can be used to create all of these artifacts.

For example when you run the following command:

```
project new --class Foo
```

You are prompted for the attributes of a new class (by default named *Foo*):

```
Object subclass: 'Foo'
  instVarNames: #()
  classVars: #()
  classInstVars: #()
  poolDictionaries: #()
  inDictionary: ''
  category: 'Foo-Core'
```

The *Foo-Core* package is created and the *Foo* class is added to the package.
The *BaselineOfFoo* is created with the following specification:

```Smalltalk
baseline: spec
  <baseline>
  spec for: #'common' do: [ spec package: 'Foo-Core' ]
``` 

For a bit more information on working with the *BaselineOf*, read the [Create Baseline][40] in the [Getting Started with GitHub document][41].
The *BaselineOfFoo* package is created for storing the *BaselineOf* on disk.

The directory `$GS_HOME/shared/repos/Foo` directory is created, a git repository is created in the directory and the *BaselineOfFoo* and *Foo-Core* packages are saved in the git repository.

Finally a *project entry* is created:

```Smalltalk
^ TDProjectSpecEntryDefinition new
    baseline: 'Foo'
      repository: 'filetree://$GS_HOME/shared/repos/Foo/repository'
      loads: #('default');
    status: #( #'active' );
    locked: true;
    yourself
```

and saved on disk as `$GS_HOME/sys/local/server/projects`.

Now is a good time make some modifications to your application class, update the `project list` command and play with some of the `project list` menu items:

![Foo project list][42]

[**COMMENTS**][28]

---
---
---

#UNDER CONSTRUCTION
---
---
---
####100. *How do I contribute to the GsDevKit_home project?*
When you decide that you would like to contribute bugfixes or improvements to the  GsDevKit_home family of projects:
 - [GsDevKit_home][15]
 - [GsDevKit_gs_client_dev][16]
 - [GsDevKit_gs_server][17]
 - [GsDevKit_todeClient][18]
 - [GsDevKit_sys_local][19]) 

or the git-based projects used by GsDevKit_home:
 - [Metacello][20]
 - [Tode][21]
 - [Ston][22]
 - [FileTree][23]
 - [Grease][24]
 - [GLASS1][25]
 - [PharoCompatibility][26]
 - [RB][27])

You start by forking the project of interest on [GitHub][14], set up your GitHub account for ssh access. 
Then you perform the following steps in your local checkout (assuming the your GitHub user-name is `dalehenrich` and the project you want to contribute to is GsDevKit_home, which you've already cloned):

```shell
git remote add dale git@github.com:dalehenrich/GsDevKit_home.git
git fetch --all
git checkout master
git checkout -b topicBranch
# do work for contribution committing along the way
git commit -m"final commit"
git push dale topicBranch
```

Then on GitHub, open a pull request from the `topicBranch` in your fork, to the `master` branch that you forked from.

---
---
---


[3]: https://github.com/jgfoster/Jade
[4]: https://github.com/glassdb/GemTools
[5]: https://github.com/GsDevKit/GsDevKit_sys_local
[6]: https://bitbucket.org/
[7]: docs/installation/installDevKitClient.md#install-client
[8]: https://code.google.com/p/glassdb/wiki/GLASSDaemonTools
[9]: http://forum.world.st/Glass-Monit-scripts-for-gemstone-td4731164.html
[10]: https://programminggems.wordpress.com/2008/09/12/slice-4/
[11]: https://kentreis.wordpress.com/2009/10/07/my-favorite-glass-front-end-server-lighttpd/
[12]: http://www.monkeysnatchbanana.com/2010/08/18/using-fastcgi-with-nginx-and-seaside/
[13]: https://git-scm.com/
[14]: https://github.com
[15]: https://github.com/GsDevKit/GsDevKit_home
[16]: https://github.com/GsDevKit/GsDevKit_gs_client_dev
[17]: https://github.com/GsDevKit/GsDevKit_gs_server
[18]: https://github.com/GsDevKit/GsDevKit_todeClient
[19]: https://github.com/GsDevKit/GsDevKit_sys_local
[20]: https://github.com/dalehenrich/metacello-work
[21]: https://github.com/dalehenrich/tode
[22]: https://github.com/GsDevKit/ston
[23]: https://github.com/dalehenrich/filetree
[24]: https://github.com/GsDevKit/Grease
[25]: https://github.com/glassdb
[26]: https://github.com/glassdb/PharoCompatibility
[27]: https://github.com/dalehenrich/rb
[28]: https://github.com/GsDevKit/GsDevKit_home/issues/new

[30]: https://downloads.gemtalksystems.com/docs/GemStone64/3.2.x/GS64-SysAdmin-3.2/9-BackupAndRestore.htm#pgfId-1069325
[31]: ../images/GsDevKit_home-pull-request.png
[32]: ../images/GsDevKit_home-pull-request-merge-detail.png
[33]: ../images/GsDevKit_home-pull-request-previous-pull-request-link.png

[40]: https://github.com/dalehenrich/metacello-work/blob/master/docs/GettingStartedWithGitHub.md#create-baseline
[41]: https://github.com/dalehenrich/metacello-work/blob/master/docs/GettingStartedWithGitHub.md#getting-started-with-github
[42]: ../images/todeFooProjectList.png