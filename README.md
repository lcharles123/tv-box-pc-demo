# tv-box-pc-demo

## About
This repository contains instructions to build deb packages and configure an apt repository providing software through package managers like APT.
This project was inteended to be used to deliver software deployment easy to TV Boxes (aka OTT boxes), but can be used in any other system
Its is higly portable since it uses containerization with Docker.

## Deb packaging

Deb packaging provides a convenient way of distributing software for debian based systems. It has automatic dependency resolution, making it easy to install, upgrade and uninstall software.

In this project we have instructions of how to produce a deb package and send it to a remote repository to be acessed using package manager.

Creation of packages are automated for many of the popular programming languages and build systems. 
In the example folder you can check examples of real programs based on:
 - C/C++
 - Java/Kotlin with maven
 - Golang
 - Javascript/Node.js #TODO: https://www.npmjs.com/package/node-deb

Each folder on this project comes with a README file with the documentation about how to construct the packages.

## Deb repository

This container comes with an http web server that servers the deb files, it uses reprepro package to automate the addition, upgrade and deletion of packages.

## Best pratices in Docker:

Docker comes to help by enabling more reusability and portability of software, the containers are defined in best pratices of conteirization, using multi-stage construction and single responsability images (#TODO).

