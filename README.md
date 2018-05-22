What if you delete a file you later realize you needed or you make a change
that doesn't turn out so good but you're many commits down the road from
the horror of these changes???

To see how to resolve this I created three text files and initialized git
on the directory.

The files initially look like this...

    file1.txt
      I am file 1

    file2.txt
      I am file 2

    file3.txt
      I am file 3


then after these commits...

    commit e5bfac1d4f1490de9a4ec8677571257502c1fa51

        added a log file and gitignore

    commit df80e8d21548e0e68e25c209306e6388e43ab656

        another good change to file2

    commit 36caed3792a1b3e65b6d767233330d43e6771d17

        a good change to file 3

    commit d9d54c03bb9b0f75333e248d2a2dbd47e9ab9d5e

        a good change to file1 and a bad change to file2

    commit 27abe8e3f7e8b1a13c95bf549f641e8ac3fd1191

        initial commit



The files look like this...

    file1.txt
      I am file 1
      a good change to file1

    file2.txt
      I am file 2
      a bad  change to file2
      another good change to file2

    file3.txt
      I am file 3
      a good change to file3

so far so good(?) but now the fun starts, I decide to delete file1 and then commit that...

    commit b07c2658149dd4d3ad59cf08728530e6e52e92be

        I just removed file1!!

and then I go on my merry way with another change to file3 so now my commit
history looks like this...

    commit 95b2606921f11884877e0e2d9f5177b2067fd1ee
        Another good change to file3

    commit b07c2658149dd4d3ad59cf08728530e6e52e92be
        I just removed file1!!

    commit e5bfac1d4f1490de9a4ec8677571257502c1fa51
        added a log file and gitignore

    commit df80e8d21548e0e68e25c209306e6388e43ab656
        another good change to file2

    commit 36caed3792a1b3e65b6d767233330d43e6771d17
        a good change to file 3

    commit d9d54c03bb9b0f75333e248d2a2dbd47e9ab9d5e
        a good change to file1 and a bad change to file2

    commit 27abe8e3f7e8b1a13c95bf549f641e8ac3fd1191
        initial commit

Now my directory look like this, NOTE: file1.txt is missing in action and...

    drwxr-xr-x   7 jrdissermac  staff   224 May 22 10:28 .
    drwxr-xr-x   7 jrdissermac  staff   224 May 22 09:20 ..
    drwxr-xr-x  12 jrdissermac  staff   384 May 22 10:35 .git
    -rw-r--r--   1 jrdissermac  staff     4 May 22 10:04 .gitignore
    -rw-r--r--   1 jrdissermac  staff    64 May 22 10:02 file2.txt
    -rw-r--r--   1 jrdissermac  staff    64 May 22 10:34 file3.txt
    -rw-r--r--   1 jrdissermac  staff  2389 May 22 10:37 log


File2.txt has a bad change...

    file2.txt
      I am file 2
      a bad  change to file2
      another good change to file2

    file3.txt
      I am file 3
      a good change to file3
      another good change to file3

And at this point I decide to push this mess up to github making a public copy
of the repo, and complicating matters...

    git remote add origin https://github.com/jdisser/gitHackery.git
    git push -u origin master

    file3.txt
      I am file 3
      a good change to file3
      another good change to file3
      yet another good change to file3.txt

So now the problem is that I need to get file1.txt back and get rid of that
bad change in file2.txt ON GITHUB!!! So this limits what can be done with
git reset since this happens in the local repo and if you try to push it git
complains that there are missing commits.

But all is not lost. *You CAN use git checkout on a single file*
instead of a whole repo. You have to checkout the commit before the deletion
and then make a commit to restore it like this...

  NOTE: you can use just a portion of the commit number about 7 characters works


    git checkout e5bfac1d4 file1.txt
    git add -A
    git commit -m "restore file1.txt using git checkout TheCommitNumber TheFileName.whatever"
    git push origin master

And to remove that "bad" change to file2 that was done long, long ago we can use
revert on just the commit that made the bad change. Revert creates an offsetting
change to the repo contents that will not break the chain of commits in the
public repo on github. So first lets see what commit we want to revert with
a git log...

    commit 0c9db28bab6c8fcc71cc5ecd37be88bba87be588
        restore file1.txt using git checkout TheCommitNumber TheFileName.whatever

    commit 39870484853a8d0c8382dbb0aef313600165fcd2
        yet another good change to file3

    commit 95b2606921f11884877e0e2d9f5177b2067fd1ee
        Another good change to file3

    commit b07c2658149dd4d3ad59cf08728530e6e52e92be
        I just removed file1gaa

    commit e5bfac1d4f1490de9a4ec8677571257502c1fa51
        added a log file and gitignore

    commit df80e8d21548e0e68e25c209306e6388e43ab656
        another good change to file2

    commit 36caed3792a1b3e65b6d767233330d43e6771d17
        a good change to file 3

    commit d9d54c03bb9b0f75333e248d2a2dbd47e9ab9d5e
        a good change to file1 and a bad change to file2

    commit 27abe8e3f7e8b1a13c95bf549f641e8ac3fd1191
        initial commit

The offending commit is d9d54c03b where a good change was made to file1 and the
bad one to file2. Reverting this commit will remove both of them but we can still
get this to work using the sorcery of git! First we'll continue with the idea
of reverting that offending commit thusly...

    git revert d9d54c03b

Yikes! I get this message...

    error: could not revert d9d54c0... a good change to file1 and a bad change to file2
    hint: after resolving the conflicts, mark the corrected paths
    hint: with 'git add <paths>' or 'git rm <paths>'
    hint: and commit the result with 'git commit'

Which is very helpful thanks to our friends at Git! So looking at file2.txt now we see

    file2.txt
      I am file 2
      <<<<<<< HEAD
      a bad  change to file2
      another good change to file2
      =======
      >>>>>>> parent of d9d54c0... a good change to file1 and a bad change to file2

so we need to edit file2.txt and then do a git add file2.txt to put it in
the working tree. After we do that our files look like this...

    file1.txt
      I am file 1

    file2.txt
      I am file 2
      another good change to file2

    file3.txt
      I am file 3
      a good change to file3
      another good change to file3
      yet another good change to file3.txt

But we're now missing the good change to file1.txt! It got reverted along with
file2.txt! But all is not lost, it's not gone in the sense it doesn't exist since
we can go back in time with our new friend

    git checkout <commit> <filename>

and after we do this...

    git checkout 36caed37 file1.txt

our files look like this...

    file1.txt
      I am file 1
      a good change to file1

    file2.txt
      I am file 2
      another good change to file2

    file3.txt
      I am file 3
      a good change to file3
      another good change to file3
      yet another good change to file3.txt

And all is again right with the world....

EPILOG

Unless you made a change in .gitignore! which will add unwanted files in the
repo! But removing stuff is way easier than restoring it!

Here's the whole git log for those of you scoring along at home, please
mark your scorecards accordingly!



    commit c598890e08242714b949cf4e4e67d1b9ccf853f9
        now restore file1.txt back to a good state using git checkout 36caed37 file1.txt

    commit 7bef7c911463ca14fd98da059a4ef1d6585d4328
        use git revert d9d54c03b to revert the bad change to file2

    commit 0c9db28bab6c8fcc71cc5ecd37be88bba87be588
        restore file1.txt using git checkout TheCommitNumber TheFileName.whatever

    commit 39870484853a8d0c8382dbb0aef313600165fcd2
        yet another good change to file3

    commit 95b2606921f11884877e0e2d9f5177b2067fd1ee
        Another good change to file3

    commit b07c2658149dd4d3ad59cf08728530e6e52e92be
        I just removed file1gaa

    commit e5bfac1d4f1490de9a4ec8677571257502c1fa51
        added a log file and gitignore

    commit df80e8d21548e0e68e25c209306e6388e43ab656
        another good change to file2

    commit 36caed3792a1b3e65b6d767233330d43e6771d17
        a good change to file 3

    commit d9d54c03bb9b0f75333e248d2a2dbd47e9ab9d5e
        a good change to file1 and a bad change to file2

    commit 27abe8e3f7e8b1a13c95bf549f641e8ac3fd1191
        initial commit
