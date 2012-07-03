# HAUNT

### Note:

This isn't done yet. And basically will only create the issue and pull request objects at this point.

### TODO
+ Decide how to mark issue has consumed (so that same issue isn't reprocessed)... should we even do this?

---

### What it is?

**Huant helps you keep your github issues under control.** It does this by allowing you to run contextual unit tests against github issues and pull requests, then make decisions based on the results about closing, sorting, tagging, and commenting.


### How it's done?

Haunt pulls all open issues/pull-requests from your repo and gathers a bunch of data about them from githubs api. It then runs a series of tests (which you define). Each test is provided a special haunt object which contains all the issue data as well as an api to act directly on the issue.


### Where to start?

You can use haunt from the command line or programatically.

##### CLI

To use Haunt from the command line, install like so:

    $ npm install haunt -g

This will give you a haunt command you can use from terminal.

    $ haunt

Running haunt with no arguments will output some simple cli documentation.

To run some tests against a repo, you might do something like this:

    $ haunt -u user:pass ./path/to/my/local/tests.js http://github.com/my/repo

**Note:** the `--user` or `-u` flag is required. We use this to authenticate against the github api. All actions performed by your tests will be made on behalf of the authenticated user.


It's also worth noting that if you don't provide a local test file, haunt will look for `haunt.js` file in the root of the repo. This might look something like:


    $ haunt -u user:pass http://github.com/my/repo

##### Programatic API

You may want to use the programatic api to build out a service, or something which routinely runs to keep your issues under control at a more consistent interval.

To use haunt programatticaly, just do something like this:

```js
    var haunt = require('haunt');

    haunt.auth('user', 'pass');
    haunt.repo('http://github.com/my/repo');

    // repo takes an optional second argument which you can use to specify
    // the path to a local test file to use instead of a remote haunt.js
    // haunt.repo('http://github.com/my/repo', './path/to/my/local/tests.js');
```


### Writing Tests

All haunt tests are assumed to be syncronous.

A basic haunt test file exports a single object with the two optional properties `pull-request` and `issue`. This looks something like this:

```js
module.exports = {

    'pull-request': { … }

    'issue': { … }

}
```

Each optional object (`pull-request` and `issue`) should include a series of tests to be ran against the specified issue type. In addition to the tests you may also specifiy a before and after property. Before will be executed before all tests are ran, after will be executed after all tests have ran. All methods are passed a single argument which contains an interface into a github issue or pull-request.

A simple issue test file might look like this:

```js
module.exports = {
    'issue': {

        'should include a <3 in the description': function (assert, haunt) {
            assert.ok(/<3/.test(haunt.description));
        },

        'after': function (haunt) {
            if (!haunt.reporter.stats.failures) haunt.tag('<3');
        }

    }
}
```

##### Issues

When made against an issue, haunt will contain the following properties.

+ issue.created_at - the created_at time of an issue
+ issue.updated_at - the updated_at time of an issue
+ issue.milestone - the assigned milestone of an issue
+ issue.assignee - the assignee of an issue
+ issue.labels - the labels for an issue
+ issue.number - the issue number
+ issue.title - the issue title
+ issue.body - the issue description body
+ issue.user - a user object for the person who filed an issue
+ issue.user.gravatar_id - a gravatar id for the user
+ issue.user.login - a user's github handle
+ issue.user.url - the url for a user's github page
+ issue.user.avatar_url - an avatar url
+ issue.user.id - a user's id number
+ issue.id - the issue's id
+ issue.comments - an array of github comment objects
+ issue.comments[x].created_at - the created_at time for a comment
+ issue.comments[x].updated_at - the updated_at time for a comment
+ issue.comments[x].user[*] - a user object for the person posting a comment (same properties as issue user obj)
+ issue.comments[x].body - the text body of a comment
+ issue.comments[x].id - the comment id
+ issue.comments[x].url - the permalink url for a particular comment


##### Pull Request

The haunt object will contain the following properties (in addition to everything included in a normal issue), when testing against a pull-request:

+ issue.diff o- the complete diff of a pull-request
+ issue.files - an array of the files changed in a commit
+ issue.files[*].patch - the patch for a specific file
+ issue.files[*].filename - the filename changed
+ issue.files[*].status - the status of the file (modified, deleted, etc.)
+ issue.files[*].changes - the number of changes made in a file
+ issue.files[*].deletions - the number of deletions made in a file
+ issue.commits - an array of git commits
+ issue.commits[*].sha - the commit sha
+ issue.commits[*].commit - a commit object
+ issue.commits[*].commit.tree - a commit tree obj
+ issue.commits[*].commit.message - the commit message
+ issue.commits[*].commit.url - permalink for a given commit
+ issue.commits[*].author - a github user object for the committing author
+ issue.commits[*].commiter - a github user object for the committer
+ issue.base - an object representing the branch the pull-request is being made into
+ issue.base.label - an object representing the branch the pull-request is being made into
+ issue.base.ref - the name of the branch being referenced
+ issue.base.sha - the sha
+ issue.base.user - the github user object who own the requesting repo
+ issue.base.repo - a github repo object
+ issue.head - an object representing the branch the the pull-request is being made from
+ issue.head.label - an object representing the branch the pull-request is being made into
+ issue.head.ref - the name of the branch being referenced
+ issue.head.sha - the sha
+ issue.head.user - the github user object who own the requesting repo
+ issue.head.repo - a github repo object


#### Before

When executed before an issue, the haunt contain everything made available to a regular issue/pull-request test.

#### After

When executed after an issue, the haunt will contain everything made available to a regular issue/pull-request test, with the addition of a mocha reporter object:

+ haunt.reporter.stats - a mocha stat object
+ haunt.reporter.stats.tests - the number of tests run
+ haunt.reporter.stats.passes - the number of tests passed
+ haunt.reporter.stats.failures - the number of tests failed
+ haunt.reporter.failures - an array of failed test object
+ haunt.reporter.failures[*].title - the title of the failed test


##### Methods

The following convenience methods are made available on all haunt objects. You can call these at any time - though I recommend you only really use then in `after` methods.

+ haunt.tag - (accepts a tagname) tags an issue/pull-request
+ haunt.close - closes an issue/pull-request
+ haunt.assign - (accepts a username) assigns an issue/pull-request
+ haunt.comment - (accepts a string) comments on an issue/pull-request
+ haunt.comment.failure - (accepts an array of failed tests) generic test failure message, which notifies a user what failed.
+ haunt.comment.warning - (accepts an array of failed tests) generic test warning message, which notifies a user what failed.

##### Examples

Here's the simple Bootstrap haunt.js file that I wrote - it saves me sooooo much time!:

```js

module.exports = {

    'pull-request': {

        'should always be made against -wip branches': function (assert, haunt) {
            assert.ok(/\-wip$/.test(haunt.branches.to.test));
        },

        'should always include a unit test if changing js files': function (assert, haunt) {
            var hasJS    = false;
            var hasTests = false;

            haunt.paths.forEach(function (path) {
                if (/\/js\/[^./]+.js/.test(path))            hasJS = true;
                if (/\/js\/test\/unit\/[^.]+.js/.test(path)) hasTests = true;
            })

            assert.ok(!hasJS || hasJS && hasTests);
        },

        'after': function (haunt) {
            if (haunt.failed.length) {
                haunt.comment.failure(haunt.failed).close();
            }
        }
    }

    'issue': {

        'should always include a tag definition': function (assert, haunt) {
            assert.ok(/tag: \w+/.test(haunt.description))
        }

        'after': function (haunt) {
            // tag as popular if > 5 +1's
            var plusOne = 0;

            haunt.comments.forEach(function (comment) {
                if (/\+1/.test(comment.text)) plusOne++;
            });

            if (plusOne > 5) haunt.tag('Popular')

            // apply user defined tags
            var tags = /tag: ([\w, ]+)/.exec(haunt.description)[1].replace(/\s+/, '').split(',');

            tags.forEach(haunt.tag);
        }


    }

}
```