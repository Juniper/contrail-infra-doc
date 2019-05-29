Debugging
=========

Change not merging
------------------

Sometimes Zuul will refuse to merge a changeset even though all required votes are present. Reasons may include merge conflict or dependency issues.

Dependencies
````````````

There are two types of changeset dependencies: implicit and explicit.

**Implicit dependency** is a dependency introduced by the git tree structure. For example, your `changeset 2001` always has some git commit (``abcd1234``).
Its parent git commit (``dede4321``) is known to gerrit as `changeset 1111`. If `changeset 1111` is not merged yet, you cannot merge `changeset 2001` - it's
a dependency. Both git commits' hashes are visible in gerrit UI.

Often you accidentaly repeat ``git commit`` while forgetting the option ``--amend``. As you issue ``git review``, it produces implicit dependencies.

**Explicit dependency** is a dependency specified in git commit message by ``Depends-On: $Change-ID``.

Unittest failures
-----------------

Logs
````
Start debugging by checking what unittests failed - you can do this in ``scons_test-FAIL.log.gz``. You can search for the tests and their results in ``job-output.txt.gz`` using the test name. 

If this doesn't help, try searching for ``scons: *`` to see all errors raised by SCons - often failures are preceded by a useful Python stacktrace.


Flakiness
`````````
We define `flakiness` as the instability of tests - some tests tend to randomly fail and finding the root cause would require a major investigation.
If you're in a hurry, just run the tests again: go to your review, open reply window and type: ``recheck``

If you suspect or have experienced some test flakiness, please let us know on #ci channel on Slack and/or contact test author.
