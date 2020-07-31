# Simple Slack Notify

[![Latest Stable Version](https://img.shields.io/packagist/v/adamkdean/simple-slack-notify?label=version)](https://packagist.org/packages/adamkdean/simple-slack-notify) [![Release date](https://img.shields.io/github/release-date/adamkdean/simple-slack-notify)](https://packagist.org/packages/adamkdean/simple-slack-notify) [![License](https://img.shields.io/packagist/l/adamkdean/simple-slack-notify)](https://packagist.org/packages/adamkdean/simple-slack-notify) [![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg)](http://standardjs.com)

Slack notification action that just works

## Introduction

I've attempted to use a few of the Slack notification actions that are currently available, but they all seem to have limitations or be quite verbose, so I set out to create a simple yet effective action that just does what you need and nothing else. In the examples below, I'll show a few different variations of how the action could be used.

The main features are:
- Status based messages meaning one step handles job successes, failures, and cancellations
- JavaScript strings for embedding environment variables or custom logic into notification strings
- Easy to add fields based on standard Slack JSON inputs

Be sure that you set the `SLACK_WEBHOOK_URL` environment variable, either in the job or in the step.

## Example usage

The simplest use would consist of relying on the webook's defaults and simply providing some text.

```
- name: Simple notification
  uses: adamkdean/simple-slack-notify@master
  with:
    text: 'This is the simplest notification'
```

Overriding the channel is sometimes needed, such as to separate out builds, deployments, and alerts perhaps.

```
- name: Channel specific notification
  uses: adamkdean/simple-slack-notify@master
  with:
    channel: '#alerts'
    text: 'Something is happening and someone should probably panic'
```

The above works well, but what would really make someone panic is if we make the alert red, right?

You can use `danger`, `warning`, `good`, or a hex code such as `#d90000`.

```
- name: Panic inducing notification
  uses: adamkdean/simple-slack-notify@master
  with:
    channel: '#alerts'
    text: 'Something is happening and someone should probably panic'
    color: 'danger'
```

Perhaps you also want to change the username?

```
- name: Panic Bot notification
  uses: adamkdean/simple-slack-notify@master
  with:
    channel: '#alerts'
    username: 'Panic Bot'
    text: 'Something is happening and someone should probably panic'
    color: 'danger'
```

The action also supports fields, but due to the limitations of GitHub actions only passing in inputs as strings, we can't use yaml arrays. So, this is how you'd specify a field:

```
- name: Specifying what to panic about notification
  uses: adamkdean/simple-slack-notify@master
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
  uses: adamkdean/simple-slack-notify@master
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
  uses: adamkdean/simple-slack-notify@master
  with:
    channel: '#example'
    text: '${env.GITHUB_WORKFLOW} (${env.GITHUB_RUN_NUMBER}) has finished'
    fields: |
      [{ "title": "Repository", "value": "${env.GITHUB_REPOSITORY}", "short": true },
       { "title": "Branch", "value": "${env.GITHUB_BRANCH}", "short": true }]
```

Now, each job has a status, which can be `success`, `failed`, or `cancelled`. Most other notification plugins use multiple blocks with `if: success()` and `if: failed()` etc but we don't need to do that. We can simply pass in the status and set status specific text. We use `if: always()` so that it runs regardless of whether the job is successful or not.

```
- name: Build notification
  if: always()
  uses: adamkdean/simple-slack-notify@master
  with:
    channel: '#builds'
    status: ${{ job.status }}
    success_text: '${env.GITHUB_WORKFLOW} (${env.GITHUB_RUN_NUMBER}) build completed successfully'
    failure_text: '${env.GITHUB_WORKFLOW} (${env.GITHUB_RUN_NUMBER}) build failed'
    cancelled_text: '${env.GITHUB_WORKFLOW} (${env.GITHUB_RUN_NUMBER}) build was cancelled'
    fields: |
      [{ "title": "Repository", "value": "${env.GITHUB_REPOSITORY}", "short": true },
       { "title": "Branch", "value": "${env.GITHUB_BRANCH}", "short": true }]
```

There are likely other ways you can use this action, so please submit a pull request if you want to add your useful example to this list. I hope this is as useful for you as it is for me.

Note: the `GITHUB_BRANCH` variable isn't standard. To get that, use the following:

```
- name: Extract branch name
  shell: bash
  run: echo "::set-env name=GITHUB_BRANCH::$(echo ${GITHUB_REF#refs/heads/} | sed 's/\//_/g')"
```

## Inputs

### `channel`

The channel you want to send to, such as '#general'

### `username`

Used to override the default username

### `color`

The color you want to use, can be good, danger, warning or a hex code

### `status`

Pass the job status through and omit color for status based color

### `text`

The message that you want to send regardless of status

### `success_text`

The message to send if status is success

### `failure_text`

The message to send if status is failure

### `cancelled_text`

The message to send if status is cancelled

### `fields`

JSON string containing an array of fields to attach to the notification

## License

MIT License

Copyright (c) 2020 Adam K Dean <adamkdean@googlemail.com>

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
