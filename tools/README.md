# Album generator using discord as a host

This site is using discord to host its albums.

To facilitate the process I create this script to upload folders of images to a dump discord channel/server, while also generating thumbnails for different sizes according to what the albums grid need, getting the AR of the shot for the mentioned grid, and creates a json file to easily use it on the jekyll site.

To use this simply create a .env file with a `DISCORD_TOKEN` with the discord bot token.