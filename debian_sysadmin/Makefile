#!/usr/bin/make -f


build: clean
	latex debian_sysadmin.tex
	dvipdf debian_sysadmin.dvi
	evince debian_sysadmin.pdf
#	evince -s tuning_pg.pdf

clean: 
	rm -f *.aux *.dvi *.log *.nav *.pdf *.snm *.toc *.out 
