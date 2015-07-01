## Notes on Contributions to Closure Library ##

Closure Library welcomes patches for features and bugfixes.


### Patches and code reviews ###

  * In most cases, changes should go through a code review.  Small patches can sometimes be attached to bug reports, but this is the exception (we need the legal formality of a CLA -- see below).
  * For a code review, upload a patch to the [Rietveld code review tool](http://codereview.appspot.com/).  The Base URL should be against the "Closure Library" codebase.  Ping the [discussion list](https://groups.google.com/group/closure-library-discuss) and we'll assign an appropriate reviewer.
  * So we know which reviews are outstanding, please add a Type-CodeReview label to an issue in the [issue tracker](http://code.google.com/p/closure-library/issues/).  If you are fixing an existing issue, you can add the Type-CodeReview label to it.  Otherwise, you can [file a new issue](http://code.google.com/p/closure-library/issues/entry). This is the only way to know which patches/reviews need attention.   Please include a link to your code review in the issue description.  Open code review requests can be tracked with a saved search:  http://goo.gl/C1gIJ
  * If you plan to add a significant component or chunk of code, it is recommended to bring it up on the [discussion list](https://groups.google.com/group/closure-library-discuss) for a design discussion before writing code.
  * If you or your organization is not listed there already, you should add an entry to the [AUTHORS file](http://code.google.com/p/closure-library/source/browse/AUTHORS) as part of your patch.




### Google Individual Contributor License ###

In all cases, contributors must sign a contributor license agreement, either for an [individual](http://code.google.com/legal/individual-cla-v1.0.html) or [corporation](http://code.google.com/legal/corporate-cla-v1.0.html), before a patch can be accepted.

### Patch ingestion ###

  * Once the code is reviewed (oftentimes, the reviewer will respond with "LGTM" for "looks good to me"), a project committer will pull the final patch and submit it to the repository.
  * Once submitted, any issues the patch fixes can be closed.