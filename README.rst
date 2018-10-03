Development guidelines
======================

This set of documentation intends to outline some rough guidelines on how software projects are to be structured and developed as part of a team at the Auckland Bioengineering Institute.
Naturally, the contents on these matters will grow over time, as contributors to this document put in their thoughts.
Also note that at this stage these are guidelines, and not hard rules set in stone that ignore the context that a project may have.
Use judgement before applying the concepts introduced in this document.

Version control
---------------

Every project must start off using a form of version control system (VCS), and for this we are using `Git <https://git-scm.com/>`__.
While `Git <https://git-scm.com/>`__ is a distributed VCS (DVCS), we do make use of `GitHub <https://github.com/>`__ as the central location for the development activities that require group coordination, as `GitHub <https://github.com/>`__ provides a number of facilities that will be outlined in this document.

As `GitHub <https://github.com/>`__ is a centralised SaaS (Software as a Service) that does not have a generally available local deployment avenue (cost is prohibitive for academic projects), `GitLab <https://about.gitlab.com/>`__ is often touted as the alternative should a local deployment be required.
This is being investigated for PMR (Physiome Mode Repository) as that project make use of `Git <https://git-scm.com/>`__ as the DVCS system and users of PMR are starting to demand features like pull requests (`GitLab <https://about.gitlab.com/>`__ uses the term "merge requests").
Regardless of the future direction, the concepts between these two competing systems are similar and should apply in most cases.

Basic Git/GitHub
----------------

In the beginning...
~~~~~~~~~~~~~~~~~~~

Every project starts off with a base commit.
While many projects do start off with a README file, some repositories do start off as a complete blank slate (`git commit --allow-empty -m 'initial commit'`).
Either way is fine, but ultimately the project **must** contain some kind of README file, using the reStructuredText markup syntax.

During prototyping
~~~~~~~~~~~~~~~~~~

Things may come up and then brought down fast, so the following sections that have many formalities may not directly apply.
However, given that prototype projects often end up in production somehow, the methodologies that were neglected will typically rear their ugly heads, so following as much of the following is highly recommended.

Commits and branches
~~~~~~~~~~~~~~~~~~~~

A project typically starts off being committed to `master`.
This is a good enough default, however that word is definitely overloaded, so a definition should be outlined somewhere to state that this is the main development branch, or the branch that is guaranteed to reference the latest stable release.
Current observation show that `master` is typically being used as the current latest development branch for many projects, and that is fine.

Once a project matures, branches should be used for introduction of features.

Pick a method, and document/stick with it for the lifetime of the project.

Best practices for Git
----------------------

Once a project is stabilised and has at least a single stable release, maintaining that stability is paramount.
For best results, this should be achieved as soon as possible, preferably as early as possible during the development stage.

Please note that some of the instructions below assumes basic knowledge
of various `Git <https://git-scm.com/>`__ terminologies.

What goes in the main branch
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The main branch is what new developers or visitors to the project page
will first see.
Depending on the nature of the project, this may be a reference to the latest release branch, or some kind of stable development branch.
This should be defined in the documentation for that given project.

Generally, this branch must have all tests passed and all documentation reviewed, as this is / may be the face of the project for the people that came in for a visit.
This also means that any commits made/pushed to this branch must be permanent - i.e. this means a forced push to the main branch (`git push -f origin <main_branch>`) **must** be avoided at all costs, to avoid randomly breaking clones/forks of the project that other developers may have.
Any changes to the main branch should be coordinated using other means.

Do note that force pushes do and will happen under other circumstances, to be documented in later sections.

When a project matures, this main branch should no longer be directly committed to, and this also means changes that were never present on some feature/development branch should also never be present here.
This is to facilitate code review and to allow better auditing and control on what commits go where.
This is especially important in the context of security related fixes - properly done, it makes applying and tracing the availability of a (security) patch for a given release branch a significantly easier task.

How to make changes
~~~~~~~~~~~~~~~~~~~

If the main branch can no longer be committed to, how can any changes be done?
This is where "feature branches" comes in.

Whenever a new feature is required or conceived, one might have accidentally started doing some development and made commits, and end up with some commits like this:

::

   40f05f3625 (master) Hooking up to the plugin framework
   d8983bda98 Improvement to that new feature, more tests
   accc552557 Adding some new foo feature
   b5d64c7ce7 (origin/master) Latest stable release

Note that `origin` here means the original source that this working tree is cloned from.

Do not fret, simply do a `git checkout -b foo_feature` and continue development for that new feature on that feature branch.

What to do with the `master` branch that is now ahead of the `origin/master`?
Make *sure* the current working tree is clean (use `git status` to check) and then simply do:

::

   git checkout master
   git reset --hard origin/master
   git checkout foo_feature        # to return to local feature branch

This cleans the working tree (wiping any changes that haven't been committed or stashed elsewhere) and then set the local `master` branch to be identical to the one in `origin`.

Naturally, with any given feature branch, it should only contain changes that relate directly to that one feature, to ease the process of auditing and reviewing of the work that is being introduced to the project.

What is in a commit
~~~~~~~~~~~~~~~~~~~

A commit (or a changeset) is the basic unit of change for a given project.
To facilitate ease of debugging, good development practices and sanity for everyone involved, there are rules that should be followed.

Imagine a commit with 20,000 lines of changes, spread throughout 60+ files, with a message that simply stated "fixed a bug".
One has to ask themselves, what bug, and huh?
While this extreme sounds comedic, in practice, this happens a lot on a much smaller scale, perhaps it would be a 200 line change with the same message.
So this is not good, what can be done?

A good start is to ensure that every commit/changeset contains only changes that relate to one specific item, where that item may be a singular sub-feature that makes up the feature, a single bug fix, or indentation/whitespace clean up, or spelling fixes to documentation (to the whole code base).
This ensures that there is good isolation between different types of changes and the different changes of the same type.
This also makes it possible/easier to review individual commits or changesets in a more focused and cohesive manner, as it reduces the amount of mental state changes in the reviewer's mind and help them focus on what actually was done.

**Example A:** the widget may now slide across the status bar due to a new method of presenting error messages presented by the API that is also being introduced in the same project.
There are at least two commits here, where the first commit should be the presentation of the new API call (or feature), and the second should be the UI changes.
If they cannot be split apart, this may be an indication of underlying architectural issues (where there may be too much tight coupling between components).

**Example B:** there may be instances where a drastic fix touch upon multiple files with maybe 100+ lines of total changes.
The commit message may be a good place to outline exactly what was done, combined with relevant comments in the code itself that hint to the reader that they should use `git blame` to find the relevant changeset that would give them the overall picture of what was done.

**Example C:** sometimes being terse isn't completely bad.
An introduction of a completely new suite of tests for the code may not need an extensive commit message.
Perhaps a single sentence will suffice when more documentation is available in the form of comments in the test cases being added.

Commit message
~~~~~~~~~~~~~~

(The `author <https://github.com/metatoaster>`__ of this section has what some might term as subjective viewpoints for this topic)

A project should adopt a specific format and then adhere to them.
For projects managed by `Tommy <https://github.com/metatoaster>`__, the commit messages follow this formula:

::

   Explanative description at most 50 characters long

   - Leave an empty line above.
   - A list of points describing what was done, and more importantly, why
     it was done, if applicable.
   - Note the indentation in point form.

     - Sometimes there may be subpoints, they be spaced out like so.
     - Even while in point form, make an attempt to form complete
       sentences.

   - All lines after the first line should be kept to a maximum width of 72
     characters.
   - These rules are in place to ensure compatible fix-width font usage
     under various different contexts.

What about actually promoting the changes into the main branch?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As a feature matures, the pull request (or merge request) should be made for this feature branch for merging into the main branch.
If the project was already set up with the process, this usually triggers the following:

- Continuous integration (CI) is triggered.
- Notification of the build failures (if any).
- Code review.
- Merge of the feature branch into the main branch at origin.

Continuous integration
~~~~~~~~~~~~~~~~~~~~~~

The first point significantly highlights why unit tests and/or integration tests should be provided with the project.
This is the very first line of defense against bad code from being accidentally added to the project itself.
While local testing may show that all is well, with an extensive test suite with a sufficiently large test matrix, issues specific to other platforms which may not have been consistently tested locally on the developer's machine will appear as a build failure on the CI platform.
Naturally, if notifications are set up, the developer responsible for the changeset will be emailed about this failure that they will then need to correct (or verify that it may be a false alarm, because of how these failures can be intermittent due to issues such as DNS).

Given that most software projects at the ABI are of the free or open source nature, there are many free-to-use services readily available to achieve CI (e.g. `Travis CI <https://travis-ci.org/>`__, `AppVeyor <https://www.appveyor.com/>`__), without having to deploy specific infrastructure such as `Buildbot <https://buildbot.net/>`__ or `Jenkins <https://jenkins.io/>`__.
While certain projects at the ABI do make use of that, the maintenance cost of these CI services may be substantial at times.
Communicate with the project lead for details if a new sub-project is to be set up under that project umbrella.

Code Review
~~~~~~~~~~~

This should happen when the commits are being finalized in the feature branch for the final merge into the main branch.
The developer should make a pull request if they have not done so already, and they may need to signal someone to be the reviewer for the feature branch.
The reviewer should go through every commit in that feature branch and inform the developer as necessary.

This is also a good time to rearrange relevant commits using `git rebase`, and the use of `git push -f` to the `fork/feature_branch` should be used to ensure that every commit/changeset is crafted to proper conveyance of the work being done.

The merger of the feature branch into the target branch
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When everything is ready, all checks are green and both the reviewer and the developer are satisfied with the work introduced in the feature branch, the feature branch may then be merged.

A goal of the final output graph should preserve all commit identifiers that was used, so they may be used in future debugging and auditing purposes.

While there are a number of strategies to doing merges, there are benefits to the one being outlined below, and the steps are:

- Ensure that the feature branch is not behind the remote target branch (or commit/changeset; more on this in a bit).
- Rebase the branch on top of that target branch; fix any conflicts, if any.
- Do not use fast-forward: merge with `git merge --no-ff`.
  Yes, while this creates a merge commit when technically none is needed, this allows the preservation of commit identifiers to be done more easily.

The target branch is often times the main development branch, however, there are situations where this is not the case.
For instance, the changes that are being made need to be backported as they may be minor fixes to a number of versions.
In this case, a common base commit to all the merge targets should be used for the rebase, before the `git merge --no-ff` is done on every afflicted branches that require the features that is being introduced.

Also note that `git rebase` may be used to execute the unit tests associated with the project for every single commit that had been made, to ensure that every commit being merged into the target branch passed all the tests.

The goal is to preserve all commit/changeset identifiers such that they are common across release branches, such that developers may be able to quickly identify whether a commit is already present in specific branches.

Testing
-------

This should be another complete topic, however this will be touched upon here as the importance of having tests relevant to each commit is extremely useful for ensuring code quality is maintained.

If a given commit has relevant tests, that the tests passed, that the tests cover all relevant lines/branches within the code being introduced, and that at the time the commit is made (and the CI being done) that all tests passed, a high degree of confidence of the quality is being conveyed by the changeset such that it can be certain that any future breakages in the code that is pinned to that commit is not directly caused by the author of the commit at the time.

The other advantage is that for any future `git bisect` being done, that the failure isn't being attributed to any existing tests that may have failed at the time of the commit, but some future changes to some package dependencies, configuration or package dependents, while at the same time, if the commits are concise and focused, the context of the change may be viewed which are all information very useful for debugging.

Good commit messages, concise and focused changesets, along with unit tests do take time to craft and create.
They may seem to be a complete waste of time now, but they should be viewed as investments to aid in solving issues with the software in the future.

Documentation guidelines
========================

Documentation should be clear, concise and ideally provide example on how the software is to be used.
Often times code changes are made without corresponding updates to the documentation.
This is problematic as end users will be confused as to why the examples do not do what they expected.
Certain languages provide frameworks (such as Python's `doctest` module) that may be leveraged to ensure that the documented examples match the implementation details (e.g. set the project up so that the CI will trigger a failure when any of the examples in the various documentation fail to execute or produce the expected output).

Naturally, the top level `README` file must contain all relevant information to get started using or developing of the project.

Spell checking is important - ensure that documentation that is being added have spelling checked against a specific dictionary (e.g. one of British English or American English, avoid mixing the two).
Ensure that sentences are grammatically correct.
The code reviewer will be of great assistance to ensure this is achieved.

Neutrality and inclusivity
--------------------------

To ensure documentation that is inclusive such that it will not make any reader feel excluded, the text should be inclusive of all gender and cultural affiliation while not excluding any person.
No favourism should be shown.
The following guidelines should help to achieve that:

- Note the usage of gender neutral language.
- Avoid the usage of any first-person, second-person pronouns except for specific contexts.

  - The usage of the singular, first-person pronouns never in the main documentation (as there may be multiple authors in a documentation, there may be ambiguities as to who may be referred to by that pronoun).
  - The usage of the singular, first-person pronoun may be used in a Frequently Asked Question context, as the usage of that pronoun refers to the reader themselves.
  - The usage of the plural, first-person pronoun (i.e. "we") may be used when referring to the owner or the owner's organisation of the project.
  - The avoidance of second-person pronouns stem from the avoidance of making assumptions about the role the current reader of the document, as often times documentation covers multiple roles.
    Use the name of the role, not the pronoun.
  - The use of second-person pronouns may be used if direct instructions to the reader is to be conveyed, or the conveyance of choice to the reader is being done.

- Do not use gendered pronouns, as it runs counter to inclusivity.
- When pronouns are required, the usage of neutral, third-person pronouns are recommended (e.g. use "one", "they", or "them", when referring to a specific, unambiguous role or group).

  - Exceptions to this can arise if a specific person is being referred; their preferred pronoun(s) should be used instead.
  - Or certain inanimate objects have specific, possibly gendered pronouns that are used.

- Address the specific role without usage of pronouns to avoid confusion between the roles being discussed within a given context.
