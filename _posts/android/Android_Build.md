title: 编译Android
categories: Android
tags: [Android]
---

# Establishing a Build Environment

	sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache libgl1-mesa-dev libxml2-utils xsltproc unzip

# Preparing to Build

## Set up environment

	. build/envsetup.sh

## Choose a target

	lunch

## make

	make -j8