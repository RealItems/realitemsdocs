---
title: 10. Configure the build scripts
tags:
  - publishing
keywords: "build scripts, generating outputs, building, publishing"
last_updated: "November 30, 2016"
summary: "You need to customize the build scripts. These script automate the publishing of your PDFs and web outputs through shell scripts on the command line."
series: "Getting Started"
weight: 10
sidebar: ridoc_sidebar
permalink: ridoc_build_scripts.html
folder: ridoc
---

{% include custom/getting_started_series.html %}

## About the build scripts

The ridoc project has 5 build scripts and a script that runs them all. These scripts will require a bit of detail to configure. Every team member who is publishing on the project should set up their folder structure in the way described here.

## Get Set Up

Your command-line terminal opens up to your user name (for example, `Users/tjohnson`). I like to put all of my projects from repositories into a subfolder under my username called "projects." This makes it easy to get to the projects from the command line. You can vary from the project organization I describe here, but following the pattern I outline will make configuration easier.

To set up your projects:

1. Set up your Jekyll theme in a folder called "docs." All of the source files for every project the team is working on should live in this directory. Most likely you already either downloaded or cloned the jekyll-documentation-theme. Just rename the folder to "docs" and move it into the projects folder as shown here.
2. In the same root directory where the docs folder is, create another directory parallel to docs called doc_outputs. 

   Thus, your folder structure should be something like this:

   ```
   projects
   - docs
   - doc_outputs
   ```

   The docs folder contains the source of all your files, while the doc_outputs contains the site outputs.

## Configure the Build Scripts

For the ridocs project, you'll see a series of build scripts for each project. There are 5 build scripts, described in the following sections. Note that you really only need to run the last one, e.g., ridoc_all.sh, because it runs all of the build scripts. But you have to make sure each script is correctly configured so that they all build successfully.

{% include tip.html content="In the descriptions of the build scripts, \"ridoc\" is used as the sample project. Substitute in whatever your real project name is." %}

### ridoc_1_multiserve_pdf.sh

Here's what this script looks like:

```
echo 'Killing all Jekyll instances'
kill -9 $(ps aux | grep '[j]ekyll' | awk '{print $2}')
clear


echo "Building PDF-friendly HTML site for ridoc Writers ..."
jekyll serve --detach --config configs/ridoc/config_writers.yml,configs/ridoc/config_writers_pdf.yml
echo "done"

echo "Building PDF-friendly HTML site for ridoc Designers ..."
jekyll serve --detach --config configs/ridoc/config_designers.yml,configs/ridoc/config_designers_pdf.yml
echo "done"

echo "All done serving up the PDF-friendly sites. Now let's generate the PDF files from these sites."
echo "Now run . ridoc_2_multibuild_pdf.sh"
```

After killing all existing Jekyll instances that may be running, this script serves up a PDF friendly version of the docs (in HTML format) at the destination specified in the configuration file.

Each of your configuration files needs to have a destination like this: `../doc_outputs/ridoc/adtruth-java`. That is, the project should build in the doc_outputs folder, in a subfolder that matches the project name.

The purpose of this script is to make a version of the HTML output that is friendly to the Prince XML PDF generator. This version of the output strips out the sidebar, topnav, and other components to just render a bare-bones HTML representation of the content.

Customize the script with your own PDF configuration file names.

### ridoc_2_multibuild_pdf.sh

Here's what this script looks like:

```
# Doc Writers
echo "Building the ridoc Writers PDF ..."
prince --javascript --input-list=../doc_outputs/ridoc/writers-pdf/prince-file-list.txt -o ridoc/files/ridoc_writers_pdf.pdf;
echo "done"

# Doc Designers
echo "Building ridoc Designers PDF ..."
prince --javascript --input-list=../doc_outputs/ridoc/designers-pdf/prince-file-list.txt -o ridoc/files/ridoc_designers_pdf.pdf;
echo "done"

echo "All done building the PDFs!"
echo "Now build the web outputs: . ridoc_3_multibuild_web.sh"
```

This script builds the PDF output using the Prince command. The script reads the location of the prince-file-list.txt file in the PDF friendly output folder (as defined in the previous script) and builds a PDF.

The Prince build command takes an input parameter (`--input-list=`) that lists where all the pages are (prince-file-list.txt), and then combines all the pages into a PDF, including cross-references and other details. The Prince build command also specifies the output folder (`-o`).

The prince-file-list.txt file (which simply contains a list of URLs to HTML pages) is generated by iterating through the table of contents (ridoc_sidebar.yml) and creating a list of URLs. You can open up prince-file-list.txt in the doc output to ensure that it has a list of absolute URLs (not relative) in order for Prince to build the PDF.

This is one way the configuration file for the PDF-friendly output differs from the HTML output. (If the PDF isn't building, it's because the prince-file-list.txt in the output is empty or it contains relative URLs.)

The Prince build script puts the output PDF into the ridoc/ridoc/files directory. Now you can reference the PDF file in your HTML site. For example, on the homepage you can allow people to download a PDF of the content at files/adtruth_dotnet_pdf.pdf.

### ridoc_3_multibuild_web.sh

Here's what this script looks like:

```
kill -9 $(ps aux | grep '[j]ekyll' | awk '{print $2}')
clear

echo "Building ridoc Writers website..."
jekyll build --config configs/doc/config_writers.yml
# jekyll serve --config configs/doc/config_writers.yml
echo "done"

echo "Building ridoc Designers website..."
jekyll build --config configs/doc/config_designers.yml
# jekyll serve --config configs/doc/config_designers.yml
echo "done"

echo "All finished building all the web outputs!!!"
echo "Now push the builds to the server with . ridoc_4_publish.sh"
```

After killing all Jekyll instances, this script builds an HTML version of the projects and puts the output into the doc_outputs folder. This is the version of the content that users will mainly navigate. Since the sites are built with relative links, you can browse to the folder on your local machine, double-click the index.html file, and see the site.

The `#` part below the `jekyll build` commands contains a serve command that is there for mere convenience in case you want to serve up just one site among many that you're building. For example, if you don't want to build everything &mdash; just one site &mdash; you might just use the serve command instead. (Anything after # in a YAML file comments out the content.)

### ridoc_4_publish.sh

Here's what this script looks like:

```
echo "remove previous directory and any subdirectories without a warning prompt"
ssh yourusername@yourdomain.com 'rm -rf /var/www/html/yourpublishingdirectory'

echo "push new content into the remote directory"
scp -r -vrC ../ridoc_outputs/doc-writers yourusername@yourdomain:/var/www/html/yourpublishingdirectory

echo "All done pushing doc outputs to the server"

```

This script assumes you're publishing content onto a Linux server.

Change `yourusername` to your own user name.

This script first removes the project folder on /var/www/html/yourpublishingdirectory site and then transfers the content from doc_outputs over to the appropriate folder in /var/www/html/yourpublishingdirectory.

Note that the delete part of the script (`rm -rf`) works really well. It annihilates a folder in a heartbeat and doesn't give you any warning prompts, so make sure you have it set up correctly.

Also, in case you haven't set up the SSH publishing without a password, see [Getting around the password prompts in SCP][ridoc_no_password_prompts_scp]. Otherwise the script will stop and ping you to enter your password for each directory it transfers.

### (Optional) Push to repositories

This script isn't included in the theme, but you might optionally decide to push the built sites into another github repository. For example, if you're using Cloud Cannon to deploy your sites, you can have Cloud Cannon read files from a specific Github repository.

Here's what this script looks like:

```
cd doc_outputs/ridoc/designers
git add --all
git commit -m "publishing latest version of docs"
git push
echo "All done pushing to Github"
echo "Here's the link to download the guides..."
cd ../../docs
```

This final script simply makes a commit into a Github repo for one of your outputs.

The doc_outputs/ridoc/designers contains the site output from ridoc, so when you push content from this folder into Github, you're actually pushing the HTML site output into Github, not the ridoc source files.

Your delivery team can also grab the site output from these repos. After downloading it, the person unzips the folder and sees the website folders inside.

### ridoc_all.sh

Here's what this script looks like:

```
. deviceinsight_1_multiserve_pdf.sh; . deviceinsight_2_multibuild_pdf.sh; . deviceinsight_3_multibuild_web.sh; . deviceinsight_4_publish.sh;
```

This script simply runs the other scripts. To sequence the commands, you just separate them with semicolons. (If you added the optional script, be sure to include it here.)

After you've configured all the scripts, you can run them all by running `. ridoc_all.sh`. You might want to run this script at lunchtime, since it may take about 10 to 20 minutes to completely build the scripts. But note that since everything is now automated, you don't have to do anything at all after executing the script. After the script finishes, everything is published and in the right location.


## Test out the scripts

After setting up and customizing the build scripts, run a few tests to make sure everything is generating correctly. Getting this part right is somewhat difficult and may likely require you to tinker around with the scripts a while before it works flawlessly.

{% include custom/getting_started_series_next.html %}

{% include links.html %}
