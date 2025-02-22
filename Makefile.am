# Main Makefile for DOSBox-X

EXTRA_DIST = autogen.sh
SUBDIRS = src include

.PHONY: dosbox-x.app

dosbox-x.app: src/dosbox-x contrib/macos/dosbox.icns src/tool/mach-o-matic
	rm -Rfv dosbox-x.app
	mkdir dosbox-x.app
	mkdir dosbox-x.app/Contents
	mkdir dosbox-x.app/Contents/MacOS
	mkdir dosbox-x.app/Contents/Resources
	mkdir dosbox-x.app/Contents/Resources/drivez
	mkdir dosbox-x.app/Contents/Resources/glshaders
	mkdir dosbox-x.app/Contents/Resources/languages
	cp -v contrib/macos/dosbox-x.plist dosbox-x.app/Contents/Info.plist
	cp -v contrib/macos/dosbox-x.pkginfo dosbox-x.app/Contents/PkgInfo
	cp -v contrib/macos/dosbox.icns dosbox-x.app/Contents/Resources/dosbox-x.icns
	cp -v contrib/fonts/FREECG98.BMP dosbox-x.app/Contents/Resources/freecg98.bmp
	cp -v contrib/fonts/wqy_1?pt.bdf dosbox-x.app/Contents/Resources/
	cp -v contrib/fonts/Nouveau_IBM.ttf dosbox-x.app/Contents/Resources/Nouveau_IBM.ttf
	cp -v contrib/fonts/SarasaGothicFixed.ttf dosbox-x.app/Contents/Resources/SarasaGothicFixed.ttf
	cp -v contrib/windows/installer/drivez_readme.txt dosbox-x.app/Contents/Resources/drivez/readme.txt
	cp -v contrib/glshaders/* dosbox-x.app/Contents/Resources/glshaders/
	cp -v contrib/translations/*/*.lng dosbox-x.app/Contents/Resources/languages/
	cp -v dosbox-x.reference.conf dosbox-x.app/Contents/Resources/
	cp -v dosbox-x.reference.full.conf dosbox-x.app/Contents/Resources/
	cp -v src/dosbox-x dosbox-x.app/Contents/MacOS/dosbox-x
# this is where it gets ugly
	codesign --remove-signature dosbox-x.app/Contents/MacOS/dosbox-x
	otool -L dosbox-x.app/Contents/MacOS/dosbox-x | grep -E '/usr/local/lib|/usr/local/opt|/usr/local/Cellar|/opt/homebrew/' | while read X; do Y=`echo "$$X" | sed -E '/^ +/s///' | cut -d ' ' -f 1`; \
		dylib=`basename $$Y`; \
		cp -v $$Y dosbox-x.app/Contents/MacOS/; \
		codesign --remove-signature "dosbox-x.app/Contents/MacOS/$$dylib" || exit 1; \
		chmod 0755 "dosbox-x.app/Contents/MacOS/$$dylib" || exit 1; \
	done
# and the libs too. NTS: Cannot use install_name_tool anymore to change anything about dylib files because that invalidates code signing of those files
	for pass in 1 2 3 4 5; do \
		for dolib in dosbox-x.app/Contents/MacOS/*.dylib; do \
			XB=`otool -L "$$dolib" | grep -E '/usr/local/lib|/usr/local/opt|/usr/local/Cellar|/opt/homebrew/' | sed -E '/^ +/s///' | cut -d ' ' -f 1`; \
			for Y in $$XB; do \
				dylib=`basename $$Y`; \
				if [ -f "dosbox-x.app/Contents/MacOS/$$dylib" ]; then true; else \
					cp -vn $$Y dosbox-x.app/Contents/MacOS/; \
					codesign --remove-signature "dosbox-x.app/Contents/MacOS/$$dylib" || exit 1; \
					chmod 0755 "dosbox-x.app/Contents/MacOS/$$dylib" || exit 1; \
				fi; \
			done; \
		done; \
	done; \
	for dylib in dosbox-x.app/Contents/MacOS/*.dylib; do \
		if [ -f "$$dylib" ]; then \
	        src/tool/mach-o-matic "$$dylib" || exit 1; \
		fi; \
	done; \
	src/tool/mach-o-matic dosbox-x.app/Contents/MacOS/dosbox-x || exit 1; \
	codesign --deep -s - dosbox-x.app/Contents/MacOS/dosbox-x || exit 1

debug-dosbox-x-app:
	(cd dosbox-x.app/Contents/MacOS && (DYLD_PRINT_LIBRARIES=1 ./dosbox-x -defaultconf -defaultdir . -silent -c 'echo hello' -c 'exit')) 2>mac-os-x-dylib-report.txt
	@echo 'Please review mac-os-x-dylib-report.txt to make sure the .app bundle loaded only from the executable path and not any external Brew libraries.'
	@echo 'This is crucial to maintain the ability to distribute this application in the Mac OS X world.'

src/tool/mach-o-matic: src/tool/mach-o-matic.cpp
	g++ -o $@ $<

clean-local:
	rm -f src/tool/mach-o-matic

contrib/macos/dosbox.icns: contrib/macos/dosbox-x.png
	rm -Rfv src/dosbox.iconset
	mkdir src/dosbox.iconset
	sips -z 16 16     contrib/macos/dosbox-x.png    --out src/dosbox.iconset/icon_16x16.png
	sips -z 32 32     contrib/macos/dosbox-x.png    --out src/dosbox.iconset/icon_16x16@2.png
	sips -z 32 32     contrib/macos/dosbox-x.png    --out src/dosbox.iconset/icon_32x32.png
	sips -z 128 128   contrib/macos/dosbox-x.png    --out src/dosbox.iconset/icon_128x128.png
	sips -z 256 256   contrib/macos/dosbox-x.png    --out src/dosbox.iconset/icon_256x256.png
	sips -z 512 512   contrib/macos/dosbox-x.png    --out src/dosbox.iconset/icon_512x512.png
	sips -z 1024 1024 contrib/macos/dosbox-x.png    --out src/dosbox.iconset/icon_1024x1024.png

	iconutil -c icns -o contrib/macos/dosbox.icns src/dosbox.iconset
	rm -Rfv src/dosbox.iconset

dox:
	rm -Rfv doxygen
	doxygen

dox-site-update:
	if [[ !( -d ../dosbox-x-gh-pages ) ]]; then git clone -b gh-pages https://github.com/joncampbell123/dosbox-x.git ../dosbox-x-gh-pages; fi
	(cd ../dosbox-x-gh-pages && rm -Rfv doxygen)
	cp -Rf doxygen ../dosbox-x-gh-pages/
	(cd ../dosbox-x-gh-pages && git add doxygen/)
	(cd ../dosbox-x-gh-pages && git commit doxygen/)

compat-site-update:
	if [[ !( -d ../dosbox-x-gh-pages ) ]]; then git clone -b gh-pages https://github.com/joncampbell123/dosbox-x.git ../dosbox-x-gh-pages; fi
	true # ----------
	(cd /mnt/main/emu/demotest && ./make-compat-chart.pl)
	cp /mnt/main/emu/demotest/compat-chart.html ../dosbox-x-gh-pages/demoscene-compat.html
	true # ----------
	(cd /mnt/main/emu/msdostest && ./make-compat-chart.pl)
	cp /mnt/main/emu/msdostest/compat-chart.html ../dosbox-x-gh-pages/msdos-compat.html
	true # ----------
	(cd ../dosbox-x-gh-pages && git add demoscene-compat.html)
	(cd ../dosbox-x-gh-pages && git add msdos-compat.html)
	(cd ../dosbox-x-gh-pages && git commit -m 'more' {msdos,demoscene}-compat.html)

install: src/dosbox-x
	mkdir -p $(DESTDIR)$(bindir)
	install -m 755 src/dosbox-x $(DESTDIR)$(bindir)
	mkdir -p $(DESTDIR)$(prefix)/share/dosbox-x/drivez
	mkdir -p $(DESTDIR)$(prefix)/share/dosbox-x/glshaders
	mkdir -p $(DESTDIR)$(prefix)/share/dosbox-x/languages
	install -m 644 $(srcdir)/CHANGELOG $(DESTDIR)$(prefix)/share/dosbox-x
	install -m 644 $(srcdir)/contrib/fonts/FREECG98.BMP $(DESTDIR)$(prefix)/share/dosbox-x
	install -m 644 $(srcdir)/dosbox-x.reference.conf $(DESTDIR)$(prefix)/share/dosbox-x
	install -m 644 $(srcdir)/dosbox-x.reference.full.conf $(DESTDIR)$(prefix)/share/dosbox-x
	install -m 644 $(srcdir)/contrib/fonts/wqy_1?pt.bdf $(DESTDIR)$(prefix)/share/dosbox-x
	install -m 644 $(srcdir)/contrib/fonts/Nouveau_IBM.ttf $(DESTDIR)$(prefix)/share/dosbox-x
	install -m 644 $(srcdir)/contrib/fonts/SarasaGothicFixed.ttf $(DESTDIR)$(prefix)/share/dosbox-x
	install -m 644 $(srcdir)/contrib/windows/installer/drivez_readme.txt $(DESTDIR)$(prefix)/share/dosbox-x/drivez/readme.txt
	install -m 644 $(srcdir)/contrib/glshaders/*.glsl $(DESTDIR)$(prefix)/share/dosbox-x/glshaders
	install -m 644 $(srcdir)/contrib/translations/*/*.lng $(DESTDIR)$(prefix)/share/dosbox-x/languages
	mkdir -p $(DESTDIR)$(prefix)/share/icons/hicolor/scalable/apps
	install -m 644 $(srcdir)/contrib/icons/dosbox-x.svg $(DESTDIR)$(prefix)/share/icons/hicolor/scalable/apps/dosbox-x.svg
	mkdir -p $(DESTDIR)$(prefix)/share/applications
	install -m 644 $(srcdir)/contrib/linux/com.dosbox_x.DOSBox-X.desktop $(DESTDIR)$(prefix)/share/applications
	mkdir -p $(DESTDIR)$(prefix)/share/metainfo
	install -m 644 $(srcdir)/contrib/linux/com.dosbox_x.DOSBox-X.metainfo.xml $(DESTDIR)$(prefix)/share/metainfo
	mkdir -p $(DESTDIR)$(mandir)/man1
	install -m 644 $(srcdir)/contrib/linux/dosbox-x.1 $(DESTDIR)$(mandir)/man1
	mkdir -p $(DESTDIR)$(prefix)/share/bash-completion/completions
	install -m 644 $(srcdir)/contrib/linux/dosbox-x $(DESTDIR)$(prefix)/share/bash-completion/completions
	-command -v setcap >/dev/null && setcap cap_net_raw=ep $(DESTDIR)$(bindir)/dosbox-x

install_strip: src/dosbox-x install
	install -m 755 -s src/dosbox-x $(DESTDIR)$(bindir)
	-command -v setcap >/dev/null && setcap cap_net_raw=ep $(DESTDIR)$(bindir)/dosbox-x

uninstall:
	rm -f $(DESTDIR)$(bindir)/dosbox-x
	rm -f $(DESTDIR)$(prefix)/share/dosbox-x/drivez/*
	rmdir $(DESTDIR)$(prefix)/share/dosbox-x/drivez
	rm -f $(DESTDIR)$(prefix)/share/dosbox-x/glshaders/*
	rmdir $(DESTDIR)$(prefix)/share/dosbox-x/glshaders
	rm -f $(DESTDIR)$(prefix)/share/dosbox-x/languages/*
	rmdir $(DESTDIR)$(prefix)/share/dosbox-x/languages
	rm -f $(DESTDIR)$(prefix)/share/dosbox-x/*
	rmdir $(DESTDIR)$(prefix)/share/dosbox-x
	rm -f $(DESTDIR)$(prefix)/share/icons/hicolor/48x48/apps/dosbox-x.png
	rm -f $(DESTDIR)$(prefix)/share/icons/hicolor/scalable/apps/dosbox-x.svg
	rm -f $(DESTDIR)$(prefix)/share/applications/dosbox-x.desktop
	rm -f $(DESTDIR)$(prefix)/share/applications/com.dosbox_x.DOSBox-X.desktop
	rm -f $(DESTDIR)$(prefix)/share/metainfo/dosbox-x.metainfo.xml
	rm -f $(DESTDIR)$(prefix)/share/metainfo/com.dosbox_x.DOSBox-X.metainfo.xml
	rm -f $(DESTDIR)$(mandir)/man1/dosbox-x.1
	rm -f $(DESTDIR)$(prefix)/share/bash-completion/completions/dosbox-x
