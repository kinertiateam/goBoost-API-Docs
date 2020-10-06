Devlopment:

Minimally, you will need the following:

Ruby >= 2.3
Bundler
NodeJS

Please note, only Linux and macOS are officially supported at this time. While slate should work on Windows, it is unsupported.

Installing Dependencies on macOS
First, install homebrew, then install xcode command line tools:

xcode-select --install
Agree to the Xcode license:

sudo xcodebuild -license
Install nodejs runtime:

brew install node
Update RubyGems and install bundler:

gem update --system
gem install bundler


to start server locally: ```bundle exec middleman server```

and you should see your docs at http://localhost:4567. Whoa! That was fast!

To Edit:
Edit the Markdown in index.html.md


To Build:
bundle exec middleman build --clean

Middleman will build your website to the build directory of your project, and you can copy those static HTML files to the server of your choice. Since you're just copying static files over, you don't need to run Middleman on your server.