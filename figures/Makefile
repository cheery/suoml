figures = biunification.png biunification_after.png

all: $(figures)

%.png: %.dot
	dot -Tpng $< -o $@
