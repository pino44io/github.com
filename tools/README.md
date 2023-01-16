# Album generator using discord as a host

This site is using discord to host its albums.

To facilitate the process I create this script to upload folders of images to a dump discord channel/server, while also generating thumbnails for different sizes according to what the albums grid need, getting the AR of the shot for the mentioned grid, and creating a json file to easily use it on the Jekyll site.

To use this simply create a .env file with a `DISCORD_TOKEN` with the discord bot token.

I recommend compressing the images with [Sqoosh](https://github.com/GoogleChromeLabs/squoosh). I personally compress my images using the `mozjpeg` compression with `quality:95`:

```
squoosh-cli --mozjpeg '{quality:95}' -d out ./
```

The difference with the lossless file is barely noticeable and it tends to reduce the image size from 50% to 75%.