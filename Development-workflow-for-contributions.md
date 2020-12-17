Contributions from users are encouraged, including simple documentation fixes, enhancement ideas, or more substantive code contributions.

Generally, the master branch is the latest stable release. The development branch contains stable, tested updates (passing local tests and integrated tests in the future as TravisCI is no longer working). Branches are used to develop and test updates before merging them into the development branch. 

Please use the following (draft) workflow if you wish to contribute. Please also refer to [this page](https://github.com/nmfs-fish-tools/Resources/blob/master/CONTRIBUTING.md) which has more guidance for productive contributions.

1. Fork the VAST repo and create a branch off of the development branch relevant to your change.
2. Make sure you can run and pass the automated tests.
3. Make changes, consistently merging in commits from the upstream development branch to keep it up-to-date.
4. Before doing a pull request, rerun the automated tests and verify they still pass. 
5. Push your local branch to your forked repo, then do a pull request to the development branch of VAST. Follow standard good practices for this.
6. Once reviewed it will be merged into development.

Started December 2020 by Cole and subject to change. Feedback welcome.