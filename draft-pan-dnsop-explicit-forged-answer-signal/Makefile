build:
	kramdown-rfc draft.md > draft.xml
	xml2rfc --html draft.xml
	
clean:
	rm draft.xml draft.html
	
install:
	sudo pacman -Sy python-build python-hatchling python-installer python-configargparse
	yay -S xml2rfc
	gem install kramdown-rfc
