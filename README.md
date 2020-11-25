# Simple Slack Notify

![GitHub release (latest by date)](https://img.shields.io/github/v/release/edge/simple-slack-notify) ![GitHub Release Date](https://img.shields.io/github/release-date/edge/simple-slack-notify) ![License](https://img.shields.io/github/license/edge/simple-slack-notify) [![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg)](http://standardjs.com)

Slack notification action that just works

## Introduction

This is the continuation of [adamkdean/simple-slack-notify](https://github.com/adamkdean/simple-slack-notify).

We've attempted to use a few of the Slack notification actions that are currently available, but they all seem to have limitations or be quite verbose, so we set out to create a simple yet effective action that just does what you need and nothing else. In the examples below, we'll show a few different variations of how the action could be used.

The main features are:
- Status based messages meaning one step handles job successes, failures, and cancellations
- JavaScript strings for embedding environment variables or custom logic into notification strings
- Easy to add fields based on standard Slack JSON inputs

Be sure that you set the `SLACK_WEBHOOK_URL` environment variable, either in the job or in the step like this:

```yaml
- uses: edge/simple-slack-notify@master
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## Example usage

The simplest use would consist of relying on the webhook's defaults and simply providing some text.

```
- name: Simple notification
  uses: edge/simple-slack-notify@master
  with:
    text: 'This is the simplest notification'
```

Overriding the channel is sometimes needed, such as to separate out builds, deployments, and alerts perhaps.

```
- name: Channel specific notification
  uses: edge/simple-slack-notify@master
  with:
    channel: '#alerts'
    text: 'Something is happening and someone should probably panic'
```

The above works well, but what would really make someone panic is if we make the alert red, right?

You can use `danger`, `warning`, `good`, or a hex code such as `#d90000`.

```
- name: Panic inducing notification
  uses: edge/simple-slack-notify@master
  with:
    channel: '#alerts'
    text: 'Something is happening and someone should probably panic'
    color: 'danger'
```

Perhaps you also want to change the username?

```
- name: Panic Bot notification
  uses: edge/simple-slack-notify@master
  with:
    channel: '#alerts'
    username: 'Panic Bot'
    text: 'Something is happening and someone should probably panic'
    color: 'danger'
```

The action also supports fields, but due to the limitations of GitHub actions only passing in inputs as strings, we can't use yaml arrays. So, this is how you'd specify a field:

```
- name: Specifying what to panic about notification
  uses: edge/simple-slack-notify@master
  with:
    channel: '#alerts'
    username: 'Panic Bot'
    text: 'Something is happening and someone should probably panic'
    color: 'danger'
    fields: |
      [{ "title": "Reason to panic", "value": "Deployed failed halfway through" }]
```

If there were multiple reasons to panic, you'd add more objects to the fields array:

```
- name: Specifying what to panic about notification
  uses: edge/simple-slack-notify@master
  with:
    channel: '#alerts'
    username: 'Panic Bot'
    text: 'Something is happening and someone should probably panic'
    color: 'danger'
    fields: |
      [{ "title": "Reason to panic", "value": "Deployed failed halfway through", "short": true },
       { "title": "Timestamp", "value": "${Date.now()}", "short": true }]
```

Did you notice that some JavaScript snook in? Input strings are evaluated as a JavaScript strings, which means you can put environment variables into your messages, such as the `GITHUB_WORKFLOW` variable or `GITHUB_RUN_NUMBER` etc. The environment is stored within the `env` variable so to access environment variables in your strings, you simply use `${env.GITHUB_REPOSITORY}` etc. Here's an example:

```
- name: Environment variable notification
  uses: edge/simple-slack-notify@master
  with:
    channel: '#example'
    text: '${env.GITHUB_WORKFLOW} (${env.GITHUB_RUN_NUMBER}) has finished'
    fields: |
      [{ "title": "Repository", "value": "${env.GITHUB_REPOSITORY}", "short": true },
       { "title": "Branch", "value": "${env.BRANCH}", "short": true }]
```

Now, each job has a status, which can be `success`, `failed`, or `cancelled`. Most other notification plugins use multiple blocks with `if: success()` and `if: failed()` etc but we don't need to do that. We can simply pass in the status and set status specific text. We use `if: always()` so that it runs regardless of whether the job is successful or not.

```
- name: Build notification
  if: always()
  uses: edge/simple-slack-notify@master
  with:
    channel: '#builds'
    status: ${{ job.status }}
    success_text: '${env.GITHUB_WORKFLOW} (${env.GITHUB_RUN_NUMBER}) build completed successfully'
    failure_text: '${env.GITHUB_WORKFLOW} (${env.GITHUB_RUN_NUMBER}) build failed'
    cancelled_text: '${env.GITHUB_WORKFLOW} (${env.GITHUB_RUN_NUMBER}) build was cancelled'
    fields: |
      [{ "title": "Repository", "value": "${env.GITHUB_REPOSITORY}", "short": true },
       { "title": "Branch", "value": "${env.BRANCH}", "short": true }]
```

There are likely other ways you can use this action, so please submit a pull request if you want to add your useful example to this list. I hope this is as useful for you as it is for me.


### Extracting GitHub branch

Note: the `BRANCH` variable isn't standard. To get that, use the following:

```
- name: Extract branch name
  shell: bash
  run: echo "::set-env name=BRANCH::$(echo ${GITHUB_REF#refs/heads/} | sed 's/\//_/g')"
```

This won't work for actions initiated by a pull request though.

### Link to run

If you want to link to the run, that's super easy. Just add the following string either to a field or to the message.

```
${env.GITHUB_SERVER_URL}/${env.GITHUB_REPOSITORY}/actions/runs/${env.GITHUB_RUN_ID}
```

So for a field you'd have:

```
{ "title": "Action URL", "value": "${env.GITHUB_SERVER_URL}/${env.GITHUB_REPOSITORY}/actions/runs/${env.GITHUB_RUN_ID}"}
```

## Inputs

| Input | Details | Example/possible values |
|:------|:--------|:------------------------|
| cancelled_text | The message to send if status is cancelled | |
| channel | The channel you want to send to | #general |
| color | The color you want to use | "good", "danger", "warning" or a hex code |
| disable_eval | Disable JS string evaluation. False by default | false |
| failure_text | The message to send if status is failure | |
| fields | JSON string containing an array of fields to attach to the notification | |
| status | Pass the job status through and omit color for status based color | |
| success_text | The message to send if status is success | |
| text | The message that you want to send regardless of status | |
| username | Used to override the default username | |

## License

```
MIT License

Copyright (c) 2020 Adam K Dean <adamkdean@googlemail.com>
                   Edge Network Technologies Ltd <core@edge.network>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
