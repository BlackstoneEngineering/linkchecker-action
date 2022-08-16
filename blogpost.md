# How to scan a website for broken links

A major problem for any documentation team is the fact that docs get updated over time, and links break. A broken link is one of the worst developer experiences. The user wants the information, they clicked on the link, they are actively interested, and then they are actively denied access. This is a terrible experience, but thankfully one that is easily avoidable. In this article, I am going to show you how to implement a link checker program to scan your website for broken links, generate a report, and with a diligent team effort, you can make broken links a thing of the past.

## A Solution
There are many tools that will scan your website for broken links. Some are free, some paid, some easy to use, some super obtuse. In this article we are going to use a python cool called [Linkchecker](https://linkchecker.github.io/linkchecker/). 

We will cover how to run a scan 
- Locally on your machine - good for one off runs. Useful to initially scope project.
- Github Action - good for periodic runs, ie weekly or monthly, with reports sent to your channel of choice (slack, zendesk, jira ...etc). 

This will enble you to run the scans manually or periodically as your needs require it. The ultimate goal is automation so that broken links are fixed as soon as they occur and become part of your teams itterative process. 

## Local

### Setup
Install the `linkchecker` program and verity it is installed.

1. Install by running `pip install linkchecker`
1. Verify install by running `linkchecker --version` . The output should look something like this: 
	```bash
	$ linkchecker --version
		LinkChecker 10.0.1 released 29.1.2021
		Copyright (C) 2000-2016 Bastian Kleineidam, 2010-2021 LinkChecker Authors
	```

### Run

To run the program we need to be aware of three things
1. The name of the program `linkchecker`
1. The output format we want to use, in this case `--output html` (other vaid output types are 'none', 'gml', 'html', 'csv', 'xml', 'failures', 'text', 'sql', 'dot', 'sitemap', 'gxml')
1. Website to scan. In this case I will scan `https://www.WebsiteToScan.com`. Note that the full `https://www` is needed. 

To run this simple scan the full command will look like the following:

```bash
linkchecker --output html http://www.WebsiteToScan.com > ./scanresults.html
```

This will run the scan and create an `scanresults.html` file with the scanlog. The html file type is the most human friendly format to parse. Please change `WebsiteToScan.com` to be your website address. 

### Fix broken links

The output file may appear confusing at first, but it's actually quite simple.

TODO: INSERT IMAGE HERE

- **URL** - This is the URL that is broken.
- **Name** - This is the link text that is shown on the page.
- **Parent URL** - This is the page where the broken link is located.

On the Parent URL page there is a broken link called Name that links to URL. Go fix it. 

To effectively fix links follow these steps
1. Open **Parent URL** page
1. Find the **Name** link on the page. I usually search the page for the **Name**
1. Replace the **URL** of the link with a correct URL. 
1. Save page and publish. 

### Make it fast

By default the linkchecker program runs with 10 threads. This can take a long time for any medium to large sized website. You can increase the number of threads all the way up to 9999, but I usually find that \~100 threads does the trick. 

Here is the command with 100 threads

```bash
linkchecker --output html --threads 100 http://www.WebsiteToScan.com > ./scanresults.html
```

### Exclude Sections of website 

Sometimes there are areas of your website that you do not want to scan. This can be anything from areas that are restricted to logged in users and thus broken to public users or areas that simply are not relevant and you dont want to scan. 

To ignore specific links you can use the `--ignore-url` subcommand. 

For example, on Atlassian confluence based websites there are a bunch of `*.action` links that are only valid for internal users, and thus will show up as broken links. 

To ignore all these type of links we can use the following command:

```bash
linkchecker \
	--output html	\
	--threads 100	\
	--ignore-url /*\.action \
	--ignore-url /anotherSection/toIgnore/* \
	http://www.WebsiteToScan.com > ./scanresults.html
```

Note the trailing `\` at the end of every line except the final line, this is necessary to expand the single command across multiple lines. 

You can use the `--ignore-url` sub-command multiple times in the same command to ignore as many different links or patterns as you want. 

### Summary

Now you have all the basics necessary to scan your website. Go ahead and give it a try. I reccomend letting this run overnight as it can sometime take hours to run on any website of significant size. Scroll down to the bottom of the output results to see how many broken links you have. Good Hunting!

## GitHub Action

Running the link checker on your local machine is good, but it is time consuming. A better solution is to run the link checker automatically and have the results become actionable for your team. In the following example I will show you how to leverage GitHub Actions to automatically run a linkchecker every month and send the results to an internal slack channel. Another good solution would be to automatically create jira tickets for each link to fix and add to your teams backlog. 

In this section I will assume you are already familiar with how GitHub actions work and are comfortable with using them. 

### Setup

Get the action code from my repo here : [BlackstoneEngineering/linkchecker-action](https://github.com/BlackstoneEngineering/linkchecker-action/blob/main/.github/workflows/check_broken_links.yaml). 

Note that there are two actions in this repo:
- `check_broken_links.yaml` - This is the action we will use in this guide. 
- `check_regex_string.yaml` - This action will search the website for a regex patten. This is useful for searching for security leaks of API Keys, or similar material in your docs. 

There are a couple of things to configure in this script before use.
1. Change the website to scan to your website
```yaml
	env:
  		WEBSITE_URL: www.YOURWEBSITEHERE.com
```
1. Modify command with ignore url's. 
```yaml
      - name: Check Links
        run: |
          linkchecker \
            --no-warnings  \
            -t 100 \
            -o html \
            --ignore-url /internal/* \
            --ignore-url /some_other_directory/~.* \
            --ignore-url /*\.extension \
            $WEBSITE_URL > ./logs/$WEBSITE_URL.html
```
1. Set the schedule. I have configured it to run every month. 
```yaml
  schedule:
    # Run on 1st day of every month
    - cron: '0 0 1 * *'
```
1. Optional - enable Slack integration. This involves uncommenting the commented out code at the bottom and adding a repo secret for `SLACK_WEBHOOK`. If enabled this will ping a slack channel called `channel_name` with the broken links as well as a link to the output results. 

Alternatively I suggest setting up an email notification or integrating with Jira issues. 

### Verify
You can verify that the link checker action works by going to your actions and manually triggering it. You should then be able to see the output results as well as the output log when it has finished running. 

## Summary

You should now be familiar with how to run the linkchecker program manually on your local machine and automatically as a GitHub Action. With this you have a no-cost, meaningful method of checking for broken links in a way that is actionable. I expect to see no more broken links on your projects going forward :-P . Go be awesome!

## Further Reading
- [Linkchecker documentation](https://linkchecker.github.io/linkchecker/)
- [BlackstoneEngineering Github Repo](https://github.com/BlackstoneEngineering/linkchecker-action)


