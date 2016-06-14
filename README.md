## README

Code & data to reproduce the analyses in the ACL 2014 paper, [Humans Require Context to Infer Ironic Intent (so Computers Probably do, too)](http://www.aclweb.org/anthology/P/P14/P14-2084.pdf), by Byron C Wallace, Do Kook Choe, Laura Kertz, and Eugene Charniak.

Made possible by support from the Army Research Office (ARO), grant 64481-MA / W9111F-13-1-0406
"Sociolinguistically Informed Natural Language Processing: Automating Irony Detection"

Contact: [Byron Wallace](mailto:byron.wallace@gmail.com)

Included files:

* The actual database - a sqlite file - is `ironate.db`
* See `irony_stats.py` for instructions on how to reproduce our analyses. This also gives examples on working with the database in python (and in SQL, since we issue queries directly). Note that this requires sklearn, numpy, & statsmodels modules to be installed.
* We are also making the data available as a simple CSV (`irony-labeled.csv`), with each row corresponding to a comment and a summary label provided. This label is 1 if any annotator (of the three) labeled any segment in the corresponding comment as 1. This is therefore a very liberal definition of 'irony', and may or may not be appropriate, depending on the aims and task.


## Database Schema

### Overview

This is documentation for the `ironate.db` file, which is a [SQLite](https://sqlite.org/) database. Basically, a SQLite database is a single file that can be queried with SQL via a SQLite engine. One such engine is the standard [command line](http://www.sqlite.org/cli.html) REPL, e.g.:

    sqlite3 ironate.db

To list all tables:

    sqlite3> .tables

To inspect the columns in a given table:

    sqlite3> .schema some_table_name

Or programatically, via your language of choice (assuming a module exists). For python, this is the `sqlite3` module, which is (happily) a built-in library. For examples of using this, see `irony_stats.py`, also in this directory.


### General notes on the database

Comments comprise segments (roughly, sentences). We do segmentation into sentences very crudely, i.e., by splitting on punctuation (".", "!", etc.). This is obviously imperfect. In any case, the actual *text* for comments is held in segments. The order field tells you where in the comment the respective segments should be. So to get back the full-text comprising a comment, you want to do something like:

    select text from irony_commentsegment where comment_id='XXX' order by segment_index;
    -- where XXX is a comment_id.

In general we collect labels at the *sentence* level. In the ACL paper we collapsed these by treating the target as "at least one sentence in the comment was labeled as ironic by someone". Also, when decisions are forced, the label is collected at the comment level: the user is asked specifically "do you think any of the sentences in this comment were intended ironically?".


### Relevant table details

`irony_comment`:

| column              | description |
|---------------------|-------------|
| `id`                | arbitrary unique identifying id |
| `reddit_id`         | unique reddit comment id |
| `subreddit`         | string capturing the subreddit to which this comment belongs |
| `thread_title`      | the associated reddit comment thread title |
| `thread_url`        | url for this thread |
| `thread_id`         | reddit thread id (string) |
| `redditor`          | unique string ID identifying the commenter (author) |
| `parent_comment_id` | this will be NULL for all top-level comments. for comments that are replies, this will be a comment identifier. |
| `to_label`          | this is a boolean indicating whether or not the corresponding comment is meant to be annotated or not. Recall that many comments will be pulled and put in the database as *contextualizing* information, i.e., we will not want to label them. |
| `downvotes`         | number of downvotes the comment had received at the time we parsed it |
| `upvotes`           | number of upvotes the comment had received at the time we parsed it |
| `permalink`         | link to this comment |

`irony_commentsegment`:

| column          | description |
|-----------------|-------------|
| `id`            | arbitrary unique segment identifier |
| `comment_id`    | the ID of the comment of which this segment is a part (see above) |
| `segment_index` | the relative location of this segment within the larger comment. This is numerical, so ordering segments by this field for a given `comment_id` produces the original comment. |
| `text`          | finally, the actual body of the segment (string) |

`irony_label`:

| column            | description |
|-------------------|-------------|
| `id`              | unique identifier for this label. |
| `segment_id`      | the segment this label refers to. This will be empty in the case of forced decision labels, which are really at the *comment* level. |
| `comment_id`      | the comment identifier for the comment segment associated with this thread (technically redundant, but convenient to have). |
| `labeler_id`      | the person who provided this label. |
| `label`           | -1 for "un-ironic", 0 for "i don't know", and 1 for "ironic" |
| `forced_decision` | this will be true when (and only when) the labeler was forced to make a decision, despite not being sure. this happens when they request additional context, for example. |
| `viewed_thread`   | boolean indicating whether the labeler viewed the thread prior to giving this label |
| `viewed_page`     | boolean indicating whether the labeler viewed the target *webpage* associated with this thread prior to giving this label. |
| `confidence`      | score on a Likert scale expressing subjective confidence in the assigned label (low, medium, high). |
