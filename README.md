# Link Checker Github Action

## Info
This repo is a example of how to use the python link-checker tool to scan a website within a github action to find broken links or to find security leaks. 

## How to use
See the scripts in the `.github/workflows` directory. 

At the top of the file insert the full url for the website you want to scan. 

```sh
env:
  WEBSITE_URL: www.YOURWEBSITEHERE.com
```

Then run the script manually from the github actions tab.

TODO: Insert Image Here

## Link Checker
The `check_broken_links.yaml` file will scan a designated website looking for links that 404 and report them. The way to use this script is to run the script and then view the artifact that is uploaded to github.

The report will look something like this: 

TODO: Image of output report here

The `Parent URL` is the page that the broken link is on. 

You can use this report to find all broken links on the site and repair them. 

I suggest setting up a Slack notification or similar so your team can get notified when broken links are found. 

### Configuration

The only configuration available is the link paths to ignore. It is common for various link paths to be either internal access only, or be valid if logged in but invalid to the general public. Links like these will be 404 to the general public, even if they are actually valid if logged in. The solution is to simply ignore those paths. 

```sh
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

Use the `--ignore-url` flag to ignore a pth. Wild cards are supported. 


## Secrets Checker

The `check_regex_string.yaml` file will scan a site and all the  content to look for a string that is matched by a regular expression. This is useful for scanning a site for secrets or API Keys or similar content. 

The way this works is by using a `config.md` file for the link checker which will log a `warning` message each time the regex string is found. Then a secondary python script parses the output to grab all the instances that match the regex. 

I strongly suggest setting up a slack notification or email blast to notify your team when a secret is leaked. 


### Configuration

Change the regular expression to match the format of the secrets / api keys your organization uses. 

```regex
warningregex=([=, ,',",:][0-9]{1,10}\/\w{80})
```
and
```python
pattern = r"([=, ,',\",:][0-9]{1,10}\/\w{80})"
````

The built in regex is for API Keys of the format `123456789/abcdefg.....` which is fairly common. Please modify this to match your use case. 