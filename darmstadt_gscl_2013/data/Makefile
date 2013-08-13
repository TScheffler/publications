##################################################################
# Variables
NORMALIZED_DIR   := normalized
UNNORMALIZED_DIR := unnormalized
IMG_DIR   := img
TMP_DIR   := tmp

##################################################################
# Targets
.PHONY: all noise_cleaner_test plot_spell \
	clean clean_noise_cleaner_test clean_plot_spell

.DELETE_ON_ERROR:

##################################################################
# all
all: noise_cleaner_test

clean: clean_noise_cleaner_test

##################################################################
# noise_cleaner_test
noise_cleaner_test: ${NORMALIZED_DIR}/1000_tweets_noise_normalization.res

${NORMALIZED_DIR}/1000_tweets_noise_normalization.res: %.res: %.in %.out
	set -e -o pipefail; \
	paste $^ | ./noise_cleaner_res > $@.tmp && mv $@.tmp $@

${NORMALIZED_DIR}/1000_tweets_noise_normalization.in: 10000_random_tweets.txt
	set -e -o pipefail; \
	./select_random_lines -n 1000 $< | awk 'NF {gsub(/\t/, " "); print}' > $@.tmp && \
	mv $@.tmp $@

${NORMALIZED_DIR}/1000_tweets_noise_normalization.out: %.out : %.in
	set -e -o pipefail; \
	character_normalizer $< | noise_cleaner -n > $@.tmp && \
	mv $@.tmp $@

clean_noise_cleaner_test:
	-rm -rf $(wildcard $(NORMALIZED_DIR)/1000_tweets_noise_normalization*)

##################################################################
# plot_spell
plot_spell: ${IMG_DIR}/treetagger_spell.png \
		${IMG_DIR}/hunspell_spell.png

${IMG_DIR}/treetagger_spell.png \
${IMG_DIR}/hunspell_spell.png: ${IMG_DIR}/%_spell.png: ${TMP_DIR}/%.slang.txt \
	${TMP_DIR}/%.nonslang.txt
	gnuplot -e "ofile='$@'" -e "slangdata='$<'" \
	-e "nonslangdata='$(word 2, $^)'" plot_spell.plg

${TMP_DIR}/%.slang.txt ${TMP_DIR}/%.nonslang.txt: ${UNNORMALIZED_DIR}/unknown_%.annotated.txt
	./spell_stat -f ${TMP_DIR}/$*.nonslang.txt < $< > ${TMP_DIR}/$*.slang.txt;

clean_plot_spell:
	-rm -rf ${IMG_DIR}/treetagger_spell.png \
	${IMG_DIR}/hunspell_spell.png $(wildcard $(TMP_DIR)/*slang.txt)
