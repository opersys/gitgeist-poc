# gitgeist proof-of-concept user manual

gitgeist is a new kind of social network that uses Git has a storage backend and transfer protocol.

Posts and comments are stored in a Git repository which is used by the engine to generate a static web page that is served over HTTP by Node.js­

## Requirements

You need a fairly recent (10.x) version of Node.js in order to use gitgeist. Recent version of Git and GPG are also recommended.

## Starting

Clone the gitgeist Git repository somewhere on your host disk. This repository contains what is required to generate a gitgeist instance 

    $ git clone 

The script directory can be directly used in your hosting directory but in order to facilate sharing the code accross instances, we recommend creating *scripts* as a symbolic link inside the .

    $ ln -s ../scripts

The scripts directory includes all the commands needed to initialize a gitgeist node.

Prior to executing the initialisation of the node Git repository, you need to make sure that you have at the GPG public key that will validate your commits on the server on which gitgeist is running.

The 'init' script will initialize a target directory with the files required for a gitgeist node to run.

    $ scripts/init

The 'init' script require 5 parameters to initialize the node

* *hostname*: The hostname of the computer on which the server will be running.
* *port*: The port number on which the node will listen.
* *email*: The email address associated with the GPG key that you want to use to validate commits uploaded to the node.
* *username*: Username to access the private section
* *password*: Password to access the private section

Finally, to start the gitgeist internal HTTP server, run the *server* script.

    $ scripts/server 

The server hosts a static HTML page and makes it accessible over HTTP at the hostname/port of your choosing.

## Posting

Clone the Git repository backing the gitgeist node storage. You can
directly clone from the Git backend through the HTTP protocol:

    git clone http://[hostname]:[port]/git gitgeist-node

You can also clone the Git repository directly if you're on the same host as the node, or you can using the Git over the SSH protocol. The way the repository is cloned does not matter.

Because the commits needs to be signed with GPG, the cloned Git repository needs to be initialised to use the right GPG key.

    $ git config user.signingkey [ID of you GPG key]
      
The ID of the GPG key is an long string of hexadecimal characters.

    % gpg --list-keys
    /home/neumann/.gnupg/pubring.kbx
    --------------------------------
    pub   rsa3072 2019-01-15 [SC] [expires: 2021-01-14]
          82B0C739018B95F302C183344D886CDB7892496A
    uid           [ultimate] François-Denis Gonthier <neumann@lostwebsite.net>
    sub   rsa3072 2019-01-15 [E] [expires: 2021-01-14]
    
The ID of the GPG key to use here is 82B0C739018B95F302C183344D886CDB7892496A

To post a new entry, you need to create a new directory in the 'posts' directory on the host. The name of the directory matters if you want your post to appear in a logical order. We recommend to prepend the directory name with a post number, ex: *0001-HelloWorld*, *0002-SecondPost*. In this example, the post numbered 0002 would appear before the post numbered 0001 on the node public page.

The post content must be be called index.md and be written in the Markdown
markup language. We use the [markdown-it](https://markdown-it.github.io/) variant of Markdown.

Once you've written you post, you need to commit it as you would do to
any other Git repositories.

    $ git add posts
    ...
    $ git -S commit -m "New post"

Each commit needs to be *signed* to be accepted by the gitgeist server. This is the reason the -S argument is used on the *git commit* command line. The GPG key used to sign the commit needs to be the private key corresponding to the key used to initialise the repository. The update Git hook will validate the signature of the commits using that key.

The next step is to send the commits to the server.

    $ git push

Once the new post gets successfully pushed, the static page will regenerate itself.

### Images

The post can include images or other binaries files. In order to be linked properly in the generated HTML code, the full path to the file, starting from the root of the Git repository must be given in the Markdown code.

For example, if you want to insert an image called image.png in your
post, here is what you must do:

    ![Image](/posts/0001/image.png)

## Following & Unfollowing

Following and unfollowing other users of gitgeist is what makes it a "social" network. When you're following another user, you will automatically get notified when that user makes change to its repository.

You can follow another gitgeist node once you know it's URL. In the root of the Git repository, a file called "following" was inserted at the initialisation of the node. This file was initialy empty but it is meant to include the list of URLs that the host follows.

The gitgeist server monitors changes to this file to track the URLs of the host it follows. Adding and removing URLs from this list will make the server follow or unfollow those URLs.

You can directly modify this file on the host. You can also remotely push commits modifying the file for the same effect.

## Commenting

*Commenting will not demanding pushing to third party Git repositories in the future. The current state of the implementation is temporary.*

Following someone gives allows you to comment on other people's post through their Git repository.

The only difference there is between creating a new posts in your own repository and commenting in a third-party repository is that, in the latter case, you are only allowed to add content in a *comments* directory under the posts you want to comment to.

Supposing http://localhost:9002 is a third party repository that I'm following. In order to comment on post *0001*, I can create a *comments* directory within the directory of that posts.

    $ git clone http://localhost:9002/git git-comments
    $ cd git-comments
    $ mkdir -p posts/0001/comments
    $ echo "hello this is a comment" > posts/0001/comments/hello.md
    $ git add posts/0001
    $ git push
    
Once you push is accepted by the server, it will appear as a comment below the post that you've selected. Comments are not ordered yet and will show up in filesystem order.

## Private section

gitgeist does not make public the posts it gathers from the repositories you follow. To view the posts of all the nodes you are following, you can access the *priv* path in your URL.

This page will be protected with basic HTTP authentication and the username and passwords that you've set when you've initialised the repository.
