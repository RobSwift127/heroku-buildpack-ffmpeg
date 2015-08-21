Heroku buildpack for ffmpeg
=======================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks) for using [ffmpeg](http://www.ffmpeg.org/) in your project.  
It doesn't do anything else, so to actually compile your app you should use [heroku-buildpack-multi](https://github.com/ddollar/heroku-buildpack-multi) to combine it with a real buildpack.

This is a static build of ffmpeg, with webm and h.264 enabled:

    mkdir ~/ffmpeg_sources
    
    cd ~/ffmpeg_sources
    curl -L --silent http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz | tar xz
    cd yasm-1.3.0
    ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin"
    make
    make install
    make distclean
    
    cd ~/ffmpeg_sources
    curl -L --silent  http://webm.googlecode.com/files/libvpx-v1.3.0.tar.bz2 | tar xjv
    cd libvpx-v1.3.0
    PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --disable-examples --disable-unit-tests
    PATH="$HOME/bin:$PATH" make
    make install
    make clean

    cd ~/ffmpeg_sources
    curl -L --silent http://download.videolan.org/pub/x264/snapshots/last_x264.tar.bz2 | tar xjv
    cd x264-snapshot*
    PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" --enable-static
    PATH="$HOME/bin:$PATH" make
    make install
    make distclean
    
    cd ~/ffmpeg_sources
    curl -L --silent http://downloads.xiph.org/releases/ogg/libogg-1.3.0.tar.gz | tar xvz
    cd libogg-1.3.0/
    PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --disable-examples --disable-unit-tests --disable-shared
    PATH="$HOME/bin:$PATH" make
    make install
    make clean
    
    cd  ~/ffmpeg_sources
    curl -L --silent http://downloads.xiph.org/releases/vorbis/libvorbis-1.3.3.tar.gz | tar xz
    cd libvorbis-1.3.3/
    PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --disable-examples --disable-unit-tests --disable-shared
    PATH="$HOME/bin:$PATH" make
    make install
    make clean
    
    cd  ~/ffmpeg_sources
    curl -L --silent http://www.libsdl.org/release/SDL-1.2.15.tar.gz | tar xz
    cd SDL-1.2.15/
    PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --disable-shared
    PATH="$HOME/bin:$PATH" make
    make install
    make clean
    
    cd ~/ffmpeg_sources
    curl -L --silent http://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2 | tar xj
    cd ffmpeg/
    PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure \
      --prefix="$HOME/ffmpeg_build" \
      --pkg-config-flags="--static" \
      --extra-cflags="-I$HOME/ffmpeg_build/include" \
      --extra-ldflags="-L$HOME/ffmpeg_build/lib" \
      --bindir="$HOME/bin" \
      --enable-gpl \
      --enable-libvorbis \
      --enable-libvpx \
      --enable-libx264 \
      --enable-static \
      --disable-shared \
      --extra-libs=-static \
      --extra-cflags=--static \
      --enable-nonfree
    PATH="$HOME/bin:$PATH" make
    make install
    make distclean
    hash -r

Usage
-----
To use this buildpack, you should prepare .buildpacks file that contains this buildpack url and your real buildpack url.  

    $ cat .buildpacks
    https://github.com/dwnld/heroku-buildpack-ffmpeg.git
    https://github.com/heroku/heroku-buildpack-ruby.git 

The first build pack is the git URL to this repo.
The second build pack is for a Rails app, see [https://github.com/heroku](https://github.com/heroku) for other app build packs.
    
    $ heroku config:set BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git

    $ git push heroku master
    ...

You can verify installing ffmpeg by following command.

    $ heroku run "ffmpeg -version"
