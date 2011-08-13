#!/usr/bin/make -f


build: clean
	latex skype_tools_pg.tex
	dvipdf skype_tools_pg.dvi
	evince skype_tools_pg.pdf
#	evince -s tuning_pg.pdf

clean: 
	rm -f *.aux *.dvi *.log *.nav *.pdf *.snm *.toc *.out 
