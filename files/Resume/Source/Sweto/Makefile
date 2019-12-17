CMD=pdflatex


all: Resume_Swetosree.pdf
Resume_Swetosree: Resume_Swetosree.pdf

%.pdf: %.tex
	$(CMD) $<

clean:
	rm -rf *.dvi  *.log *.ps *.aux *.out 
remjunk:
	rm -rf *.dvi  *.log *.ps *.aux *.out 
