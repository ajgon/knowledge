# Download music from YouTube for Plex

* Install dependencies:

* Download music track from YouTube:
    * `youtube-dl -f 140 "https://www.youtube.com/watch?v=MkNeIUgNPQ8"`
* Convert downloaded track to mp3:
    * `avconv -i Adventures\ -\ A\ Himitsu\ \(No\ Copyright\ Music\)-MkNeIUgNPQ8.m4a Adventures\ -\ A\ Himitsu.mp3`
* Set up mp3 tags properly:
    * `mid3v2 -a Adventures -t "A Himitsu" -A "Album with No Copyright Music" -T "1/11" Adventures\ -\ A\ Himitsu.mp3`
        * `-a` - artist
        * `-t` - title
        * `-A` - album
        * `-T` - track number/all tracks
* Remove old `*.m4a` file:
    * `rm -rf Adventures\ -\ A\ Himitsu\ \(No\ Copyright\ Music\)-MkNeIUgNPQ8.m4a`
