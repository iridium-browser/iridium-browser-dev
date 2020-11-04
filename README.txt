https://github.com/iridium-browser/iridium-browser/blob/master/README.md

As for installing dependencies, you may be able to ask the package manager to
install the BuildRequires from any repository package (chromium, or
iridium-browser itself if you have the repository added to the system):

openSUSE:
	zypper si -d chromium
