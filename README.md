# investigate_cmake_superbuild
This repo is for my exploration into what a CMake superbuild is.

It seems like a superbuild is a CMake project that only builds other CMake projects.

## ExternalProject vs FetchContent
There seems to be an older style where it only uses [ExternalProject_add](https://cmake.org/cmake/help/latest/module/ExternalProject.html) to do so.
Newer stuff can supposedly use [FetchContent](https://cmake.org/cmake/help/latest/module/FetchContent.html), but that has some significant differences.

`ExternalProject_add` runs at build time and seems to isolate the builds in the binary directory.
It can be used to build any kind of project, not just ones built with CMake
`FetchContent` downloads source at configure time and then can be "made available" via [add_subdirectory](https://cmake.org/cmake/help/latest/command/add_subdirectory.html).
That means a FetchContent based superbuild can only work with other CMake projects, and ones that play nicely with the pattern at that.

There seems to be a bunch of requirements to play nicely with `FetchContent` based superbuilds.
There's some talk about a hypothetical package management solution somewhere for CMake that details it.
One thing is to alias targets such that they can be used from the building project the same way as if `find_package()`'d.
Another seems to be the ability to disable the install target of a subproject.
I saw more discussion about how rpath setting could get screwed up.
As far as I can guess, using FetchContent's advantage is that targets being built in the same project allows building some targets without building all of them.
Maybe it allows more efficient parallelizing of builds.

I bet [CPM CMake](https://github.com/cpm-cmake/CPM.cmake) would be a great tool in a FetchContent based super build.

## Links that talk about superbuilds

* 2021-06-25 https://www.scivision.dev/cmake-fetchcontent-vs-external-project/ - a blog talking about FetchContent and ExternalProject
* 2021-04-21 https://www.reddit.com/r/cmake/comments/mvsuun/cmake_superbuild_pattern/ - a user recommends against superbuilds, in favor of FetchContent
* 2020-12-14 https://www.youtube.com/watch?v=nBptg3SHPGU  a youtube video about superbuilds - author says they're useful for packaging fixed versions of things but not so great for daily development (timestamp 13:15)
* 2020-12-13 https://gitlab.kitware.com/cmake/cmake/-/issues/21592
* 2020-06-10 https://discourse.cmake.org/t/correct-way-to-use-third-party-libraries-in-cmake-project/1368/2 A CMake developer ranking superbuild's highly for using third party libraries, while also warning that they're fragile and can make development harder.
* 2018-09-20 (last edit) https://github.com/dev-cafe/cmake-cookbook/tree/v1.0/chapter-08 these are examples from an oreilly book meant to introduce the reader to the superbuild pattern
* 2018-03-16 https://www.reddit.com/r/cmake/comments/84zou4/superbuildexternal_projects/ - someone looking for alternatives to the superbuild pattern
* 2017-12-08 https://www.kitware.com/cmake-superbuilds-git-submodules/ 


## Examples of superbuilds
* https://github.com/OpenChemistry/openchemistry
* https://github.com/robotology/robotology-superbuild
* https://simpleitk.readthedocs.io/en/master/building.html#building-using-superbuild


## Alternatives to superbuilds

### Use a tool that will call a bunch of projects in the right order:

* [colcon](https://colcon.readthedocs.io)
* [catkin_make or catkin_make_isolated](https://github.com/ros/catkin)
* [catkin_tools (which has a CLI tool named catkin)](https://github.com/catkin/catkin_tools)

### Write a bunch of scripts to build projects in the right order

I remember the [Ignition Robotics](https://www.ignitionrobotics.org) used to be that way, but now I think it uses colcon.

### Vendor your packages
This means having just a CMake package that builds its dependencies from source along with it's own targets.

* Maybe by calling FetchContent and add_subdirectory
  * Maybe with help of another tool like [CPM](https://github.com/cpm-cmake/CPM.cmake).
* Maybe with git submodules and custom CMakeLists.txt
* Maybe by having a step to download binaries or static libs and building against that
* Maybe by copy/pasting the code into your repo (for header only libraries)

### Monorepo

Ugh.
This is fine if the output of the project is an executable that no one every needs to link against or use in the process of building other software.
Like making a game - without support for modding - that's a great place for a monorepo.
In any other context a monorepo is a massive pain for users.
