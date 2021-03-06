# Releasing Swift Protobuf

---

When doing a release:

1. Examine what has changed

   Github's compare UI does a reasonable job here.  [Open that UI](https://github.com/apple/swift-protobuf/compare)
   and set the _base_ to be the previous tag (_vX.Y.Z_), and the _compare_ can be left at _master_
   since that is what the release is cut off of.

   It usually works best to open the links for each commit you want to look at in a new browser
   window/tab.  That way you can review each independently rather then looking at the unified
   diffs.

   When looking at a individual commit, at the top github will show that it was a commit on master
   and include a reference '#XYZ', this tells you what pull request it was part of.  This is useful
   for doing the release notes in the next section.

1. Validate the versions numbers

   1. Inspect `Sources/SwiftProtobuf/Version.swift` and ensure the number is what you expect for
      the next release.

      Normally we try to bump _master_ to a new revision after each release, so this number may
      be fine, but if this release includes import things (api changes, etc.), it may make sense
      to go back and bump the _major_ or _minor_ number instead.  If you do need to change the
      number: `DevTools/LibraryVersions.py a.b.c`.

   1. Run our tool to ensure the versions are all in sync:

      ```
      $ DevTools/LibraryVersions.py --validate
      ```

      This will run silently if everything was ok; if something was wrong, you'll need to figure
      out why and get things back in sync.

1. Create a release on github

   Top left of the [project's releases page](https://github.com/apple/swift-protobuf/releases)
   is _Draft a new release_.

   The tag should be `v[a.b.c]` where the number *exactly* matches one you examined in
   `Sources/SwiftProtobuf/Version.swift`.

   For the description call out any major things in that release.  Usually a short summary and
   then a reference to the pull request for more info is enough.

1. Publish the CocoaPod

   _Note:_ The `pod` binary is currently pushing the use of the `.swift-version` file to control
   how a spec is checked for `spec lint` and `trunk push`. But the file isn't well documented and
   there has been some questions around that net that seem to imply people interpret this as the
   "version of Swift the pod supports". Since SwiftProtobuf supports multiple versions, a
   `.swift-version` file currently isn't included. So you want the `spec lint` to pass with only
   the warning about the file. And when doing the `trunk push` you'll want to include
   `--allow-warnings` to let the push happen despite the warning. Even if you use
   `--swift-version=#` to either commend, the `pod` binary still produces the warning, so it
   doesn't work to avoid the warning.

   1. Do a final sanity check that CocoaPods is happy with the release you just made, in the project
      directory:

      ```
      $ pod spec lint SwiftProtobuf.podspec
      ```

      _Note:_ This uses that local copy of the podspec, but pulls the code off the tag on github.

      If this doesn't pass, you have two options:

      - If the problem is just with the `podspec`, you can edit it, and try again.  The version of
        the podspec in the branch doesn't really matter, so just ensure things get fixed on master
        for the future.
      - If the problem is within the code, you'll likely need to abandon this release.  Fix the
        code and start the process over cutting a new tag.

   1. Publish the pod:

      ```
      $ pod trunk push [--allow-warnings] SwiftProtobuf.podspec
      ```

      _Note:_ This uses that local copy of the podspec.

      See the _Note_ at the start of this section about `.swift-version` and why you might
      need `--allow-warnings`.

1. Bump the version on _master_

   To help tell if someone is using _master_ before it has been cut as a release, go ahead and
   bump it to a new revision:

   ```
   $ DevTools/LibraryVersions.py [a.b.c]
   ```

   Where _c_ is one higher than the release that was just done.

   Make sure to commit/merge this _master_ on github.

