[![Actions Status](https://github.com/gfaraj/super-bot/workflows/Node%20CI/badge.svg)](https://github.com/gfaraj/super-bot/actions)


# super-bot
A simple but extensible bot written in Node.js. Currently there are Whatsapp, Slack, and Discord interfaces, but any messaging platform can be supported by creating a client for it (e.g FB Messenger, MS Teams, IRC).

## Docker

Running the bot service as a docker container is not yet supported due to incompatibility issues with the database provider. The bot clients are all dockerized, though, and [here is a handy docker-compose file](https://gist.github.com/gfaraj/e9459a5d90cdcb923655f70ae456b96a) to quickly start all of them.

## Installing from source

Clone this repository:

```
git clone https://github.com/gfaraj/super-bot.git
```

and install its dependencies by running:

```
npm install
```

Make sure you have npm and Node 10 or newer installed.

## Starting the bot

You can run the bot service with the following command:

```
npm run start
```

The bot service starts a web server on port 3000 (which is configurable) and accepts POST requests on "/message" with the body being JSON data representing a message object. The bot then will run that message through any commands that are supported and sends back the response. This makes it really easy to have any number of interfaces tied to the same bot service. Currently there's no security to restrict access to this endpoint so be aware that anyone could post messages to the bot.

## Configuration

The bot uses a JSON configuration file located in the ./config folder. See the [config](https://docs.npmjs.com/cli/config) package documentation for more information. The Whatsapp client is also configured this way.

## Clients

The same bot service can be utilized through different clients. The clients are how users interface with the bot service. They handle all the platform-specific tasks like listening for new messages and transforming them into a standard message object for the bot service.

| Repo | Description
|--- |---
| [super-bot-whatsapp](https://github.com/gfaraj/super-bot-whatsapp) | Chat interface for Whatsapp
| [super-bot-slack](https://github.com/gfaraj/super-bot-slack) | Chat interface for Slack
| [super-bot-discord](https://github.com/gfaraj/super-bot-discord) | Chat interface for Discord

## Plugins

The bot is driven by plugins. Each plugin can define any number of commands that it supports (a command can only be supported by a single plugin). A plugin can also subscribe to be called whenever a message has not been handled by any command (for example, as a fallback). It's required that a plugin export a default function that will be called during initialization.

```
export default function(bot) {
    bot.command('echo', (bot, message) => {
        bot.respond({ text : message.text, attachment : message.attachment });   // just respond back with the same message.
    });
    bot.raw((bot, message, next) => {
        if (message.text.includes('foo')) {  // check if we can handle this raw message.
            bot.respond('bar');
        }
        else {
            next();  // call next if your plugin can't handle this message.
        }
    });
}
```

### record

This plugin allows you (and your friends) to record key-value pairs with optional attachments (currently only images and Whatsapp stickers are supported). The recordings are scoped per chat. There is a plan to support global recordings in the future.

```
record <name> <value>
```

The value is optional if an attachment is provided.

The plugin also registers the "forget" command to remove a recording. Only you or the recording's author are allowed to do this. It also registers the "recordings" command which sends a list of all existing recordings that match a given string (or all if no string is provided).

Examples:
```
record hi Hello everyone!
```
After doing that, when the bot is given the command "hi" it will respond with "Hello everyone!".
```
(as a reply to an image message)
record kids
```
The Whatsapp client will append any replied message to the current command, so in the example above it will save a "kids" recording with the image attachment. Then when the bot is given the "kids" command, it will send that image.

```
record stocks !google stock msft
```
This is an unofficial way to write shortcuts to other commands. After the command above, if given the "stocks" command, the bot will respond with "!google stock msft" and it will then respond to that command. In the future, a proper command chaining feature based on piping is planned.

### translate

This plugin translates a given text into a target language like so:

```
translate es Hi, how are you doing?
```
It takes a two-letter locale as the first parameter and the text to translate after it.

This uses the Google Translate API to perform the translation. You need to [set up a Google Cloud project](https://cloud.google.com/translate/docs/quickstart-client-libraries#client-libraries-usage-nodejs) with Translate support to be able to use this plugin. There is a cost to this if the free quotas are exceeded, so be mindful of that.

### google

This plugin will perform a Google search and return the first 2 results. It's currently scraping the google search results (using puppeteer to process and render the initial web page, then using cheerio to grab the information). This is not permitted by Google so use at your own risk. 
```
google game of thrones
```
The response is something like:
```
1) https://www.hbo.com/game-of-thrones
2) https://en.wikipedia.org/wiki/Game_of_Thrones
```

This plugin also exposes a "gimg" command that performs a Google Image Search and sends back the first image result (sends the actual image, not just a link). Currently it's returning a low-res image but it's possible to adjust it to get the original image.

### evaluate

This plugin will evaluate an expression and respond with the result. It exposes two commands, eval and calc. The eval command accepts any type of (limited - it uses safe-eval) javascript expression, while the calc command will only work with numeric expressions.

```
eval 'Hello' + ' world!'
calc 10 + 20 / 2
calc 15 - sqrt(4)
```
Responds with:
```
Hello world!
20
13
```

### sticker

This plugin provides utilities related to Whatsapp stickers. The "stickerize" command will take any image and convert it to a Whatsapp sticker.

```
(as a reply to an image message)
stickerize
```
This would send a message with the quoted image as a sticker.

### code

This plugin allows you to build and run code in many languages. It will respond with any build errors or any output generated by your program. You can use this plugin like this:

`!code -l<language> <code text>`

Examples:

`!code -lc# public static class P { public static void Main() { System.Console.WriteLine("Hello, world!\n"); } }`

`!code -lgo package main
import "fmt"
func main() {
  fmt.Printf(''Have fun!")
}`

There are a couple of shortcuts for C# and C++ currently and more can be added easily:

`!cs public static class P { public static void Main() { System.Console.WriteLine("Hello, world!\n"); } }`

```
!c++ #include<iostream>
int main() { std::cout << "Hey there!\n"; }
```

The plugin also supports passing in a code file as an attachment.

### Other commands

There are many other commands available like remindme, trivia, poll, factoid, and several imaging utilities.

## Contribution

Contribution of any kind is welcome! Please feel free to create your own plugins for the bot service, chat interfaces, features or bug fixes. Fork the repository, create your own branch and submit pull requests.

# Disclaimer

This project was done for educational purposes. This code is in no way affiliated with, authorized, maintained, sponsored or endorsed by WhatsApp or any of its affiliates or subsidiaries. This is an independent and unofficial software. Use at your own risk.
