Outlined below are suggested best practices when it comes to
using git and GitHub PRs so that fellow developers can use to
understand changes to the code base. A rather complete tutorial
on git can be found on RIP tutorial.

PRs are Peer Reviews as well as Pull Requests
=============================================

#. They need to be reviewed by at least another person. Code is a form of documentation and it needs to be understood to prevent silo-ing. The more people that see the changes better more people understand the changes. This is a chance to talk about code techniques, new patterns, tips and tricks, etc.

   #. Given enough eyeballs, all bugs are shallow. This also helps clean up trivial mistakes and helps all developers learn what is expected of our code.

   #. Use the Review Requested page on GitHub to see all pending review requests.

#. When giving Peer Reviews, be earnest and kind.

   #. Read through the commits rather than just looking at all changes. Use the commit messages to as a guide to understanding the changes.

   #. Respect that all the developers here are talented and hardworking but come from different backgrounds. A few trivial mistakes are to be expected and outright misunderstandings are opportunities for teaching better ways.

#. Use fixup and squash commits with autosquash

   #. If you’re just addressing comments to your PR, commit the changes to your branch with a fixup or squash branch that will be handled later. This makes it easy to see the commit that addresses a specific issue in the PR. Overview of the fixup/autosquash workflow.

#. Commits should be small, but still enough code to complete an idea.

   #. The change sets that a fellow developer needs to understand are more comprehensible when served up in bite sized chunks

   #. If there’s a need for bullet points in a commit message or lots of ‘ands’, consider breaking the commit up with each bullet point having it’s own commit.

   #. Commits represent waypoints/savepoints in local development. If something gets too messed up (renamed a lot of files the wrong way), just reset to that commit.

#. Respect fellow developers by respecting the commit messages, they ARE important as they are the first set of documentation other developers see about the code

   #. The Linus Torvalds standard

      #. Further explanation for Linus' standard (summarized below, with links)

         #. Separate subject from body with a blank line

         #. Limit the subject line to 50 characters

         #. Capitalize the subject line

         #. Do not end the subject line with a period

         #. Use the imperative mood in the subject line

         #. Wrap the body at 72 characters

         #. Use the body to explain what and why vs. how

      #. Some editors do this formatting automatically if used to create the commit message (emacs, vim, nano (to some extent))

      #. Keep code formatting and other code minutiae in separate commits from actual code changes. Again this is to help make it easy for fellow developers to understand the actual changes and not have to wade through trivialities amongst actual changes.

      #. It’s OK to have lots of commits for a PR so long as they are small with good commit messages as it is easier to understand the flow the code changes represent.
