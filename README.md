# Steemvote

This is a program that votes for posts on Steem. It uses piston and thus requires Python 3.

The included script, `steemvoter`, works by monitoring all comments on Steem.
If a comment is eligible for voting, it is stored in a database. The eligible comments
are periodically pulled from the database and voted on after they are a pre-set age.

## Installation

```
$ sudo python3 setup.py install
```

The above command will install all required dependencies.
You can run steemvoter like so: `steemvoter -c path/to/config.json`.

## Configuration

A configuration file specifies the authors to vote on, the key to sign votes with, etc.
Your configuration file can be supplied with a command-line argument (`steemvoter -c path/to/config.json`).
If not supplied, steemvoter will look for the file `steemvote-config.json` or `steemvote-config.yaml` in the current directory.

Example configuration files are located in the [example-configs](./example-configs) directory.

The following config values are required:

- `authors`: A list of authors, and optional values for each (see below).
- `voter_account_name`: The name of your account.
- `vote_key`: The private key used to sign votes with (This should be your "Posting" key).

### Authors

The `authors` list in your config file contains a list of objects.
Each object needs a `name` value, which is the name of the author.

Optional values for each author:

- `priority`: The priority (explained below) that the author has (Default: `normal`).
- `vote_replies`: Whether to vote on all replies the author makes, in addition to their regular posts (Default: `false`).
- `weight`: The weight to vote with. `100.0` represents an upvote, and `-100.0` represents a downvote (Default: `100.0`).

*NOTE* that if you just want to add an author and use the default values, a string can be used instead of an object.

### Delegates

Delegates are other Steem accounts. Steemvoter will watch for votes that your delegates cast,
and will vote for the comments that they vote on.

The `delegates` config list is similar to the `authors` list. However, delegates do not have a `vote_replies` value.

Optional values for each delegate:

- `priority`: The priority (explained below) that comments this delegate votes for have (Default: `normal`).
- `weight`: The weight to vote with. `100.0` represents an upvote, and `-100.0` represents a downvote (Default: `100.0`).

Also like authors, you can use a string instead of an object and the default optional values will be filled in for you.

### Voting Rules

These voting rules are optional, and specify how to vote.

The value `blacklist_categories` can be specified with a list of categories. Comments in those categories will
never be voted for (Default: `[spam]`). Likewise, the value `blacklist_authors` can be specified with a list of author names.
Posts by those authors will never be voted for, even if delegates vote for them.

The following values can be specified as seconds or as human-readable strings (e.g. `"max_post_age": "2 days"`):

- `max_post_age`: The maximum age a post can be for it to be voted on (Default: `2 days`).
- `min_post_age`: The age a post must be before it is voted on (Default: `1 minute`).
- `vote_interval`: The timespan that steemvoter waits between collecting and voting on eligible comments (Default: `10 seconds`).

The following values can be specified as decimals or as human-readable percentages (e.g. `"priority_high": "75%"`):

- `priority_high`: The minimum fraction of voting power that you must have to vote for high priority comments (Default: `80%`).
- `priority_normal`: The minimum fraction of voting power that you must have to vote for normal priority comments (Default: `90%`).
- `priority_low`: The minimum fraction of voting power that you must have to vote for low priority comments (Default: `95%`).

### Database

Steemvoter uses a sqlite database to store data.

By default, the database will be a file called `database.db` in the current directory.
This behavior can be changed using the `database_path` config value.

### RPC

If no RPC options are specified, a public node will be used.

Available RPC options:

- `rpc_node`: The URL of the node to connect to.
- `rpc_user`: RPC username to use.
- `rpc_pass`: RPC password to use.

## Example Configurations

Upvote every post by [@klye](http://steemit.com/@klye), including his replies to other posts.
Wait until posts are 30 seconds old to upvote them.

```
{
    "authors": [
        {"name": "klye", "vote_replies": true}
    ],
    "vote_key": "5XX...",
    "voter_account_name": "myname",
    "min_post_age": 30
}
```

Upvote the posts by [@klye](http://steemit.com/@klye), but not his replies to other posts.
Do not vote if his post is more than 2 hours old.

```
{
    "authors": [
        "klye"
    ],
    "max_post_age": "2 hours",
    "vote_key": "5XX...",
    "voter_account_name": "myname"
}
```
