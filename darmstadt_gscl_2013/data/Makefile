##################################################################
# Variables
NORMALIZED_DIR   := normalized
UNNORMALIZED_DIR := unnormalized
IMG_DIR   := img
TMP_DIR   := tmp

##################################################################
# Targets
.PHONY: all noise_cleaner_test plot_spell test test_prf test_bleu \
	clean clean_noise_cleaner_test clean_plot_spell clean_test

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

##################################################################
# test
EVAL_DIR     := evaluation
EVAL_TMP_DIR := ${EVAL_DIR}/tmp
EVAL_SCRIPT_DIR := ${EVAL_DIR}/scripts
EVAL_DIR     := evaluation

EVAL_CORPUS       := ${EVAL_DIR}/corpus
EVAL_SRC_CORPUS   := ${EVAL_TMP_DIR}/src_corpus.txt
EVAL_SRC_SGML     := ${EVAL_TMP_DIR}/src_corpus.sgml
EVAL_SRC_SELFTEST_SGML := ${EVAL_TMP_DIR}/src_selftest_corpus.sgml
EVAL_REF_CORPUS   := ${EVAL_TMP_DIR}/ref_corpus.txt
EVAL_REF_SGML     := ${EVAL_TMP_DIR}/ref_corpus.sgml
EVAL_TRNSL_CORPUS := ${EVAL_TMP_DIR}/trns_corpus.txt
EVAL_TRNSL_SGML   := ${EVAL_TMP_DIR}/trns_corpus.sgml

RESTORE_LINE = awk '/^[[:blank:]]*<\/message/{printf("\n"); next} /<\/sentence>/{printf(" "); next} NF{printf("%s ", tolower($$0))}'

STRIP_MARKS = awk -F '\t' '/^${REGEXP}:/ {print gensub(/<<<(([^>]|>[^>])+)>>>/, "\\1", "g", $$2) \
	"\n</message>"}'

NORMALIZATION_PROCEDURE = misspelling_restorer | umlaut_restorer | \
	slang_normalizer | sentence_splitter -x | tokenizer -x | character_squeezer -1

define MAKE_CORPUS
	set -e -o pipefail; \
	 ${STRIP_MARKS} $< | sentence_splitter -x | tokenizer -x | \
	${RESTORE_LINE} > $@.tmp && mv $@.tmp $@
endef

define MAKE_SGML
	set -e -o pipefail; \
	awk -F '\t' 'BEGIN {print "<${CORPUS_NAME} trglang=\"de\" srclang=\"any\" setid=\"twitter-2013\" >\n\
            <doc docid=\"twitter-2013\" genre=\"twitter texts\" origlang=\"cs\" sysid=\"normalization\">"} \
	NF {print "<seg id=\"" ++i "\">\n" $$0 "\n</seg>"} \
	END {print "</doc></${CORPUS_NAME}>"}' $< > $@.tmp && \
	mv $@.tmp $@
endef

test: test_prf test_bleu test_bleu_self

##################################################################
# BLEU
BLEU_SCRIPT := ${EVAL_DIR}/scripts/mteval-v13.pl --international-tokenization -b

test_bleu: ${EVAL_SRC_SGML} ${EVAL_TRNSL_SGML} ${EVAL_REF_SGML}
	${BLEU_SCRIPT} -s ${EVAL_SRC_SGML} -t ${EVAL_TRNSL_SGML} -r ${EVAL_REF_SGML}

test_bleu_self: ${EVAL_SRC_SGML} ${EVAL_SRC_SELFTEST_SGML} ${EVAL_REF_SGML}
	${BLEU_SCRIPT} -s ${EVAL_SRC_SGML} -t ${EVAL_SRC_SELFTEST_SGML} -r ${EVAL_REF_SGML}

# source file for translation
${EVAL_SRC_SGML}: CORPUS_NAME := srcset
${EVAL_SRC_SGML}: ${EVAL_SRC_CORPUS}
	 ${MAKE_SGML}

${EVAL_TRNSL_CORPUS} ${EVAL_SRC_CORPUS}: REGEXP := OrigMarked
${EVAL_SRC_CORPUS}: ${EVAL_CORPUS}
	 ${MAKE_CORPUS}

${EVAL_SRC_SELFTEST_SGML}: ${EVAL_SRC_SGML}
	set -e -o pipefail ; \
	awk -v IGNORECASE=1 'NF {sub(/srcset/, "tstset", $$0); sub(/sysid="[^"]"/, sysid="selftest"); print}' $< > $@.tmp && \
	mv $@.tmp $@

# reference file for translation
${EVAL_REF_SGML}: CORPUS_NAME := refset
${EVAL_REF_SGML}: ${EVAL_REF_CORPUS}
	 ${MAKE_SGML}

${EVAL_REF_CORPUS}: REGEXP := CorrectMarked
${EVAL_REF_CORPUS}: ${EVAL_CORPUS}
	 ${MAKE_CORPUS}

# translation file
${EVAL_TRNSL_SGML}: CORPUS_NAME := tstset
${EVAL_TRNSL_SGML}: ${EVAL_TRNSL_CORPUS} ${EVAL_SRC_SGML}
	${MAKE_SGML}

${EVAL_TRNSL_CORPUS}: ${EVAL_CORPUS}
	set -e -o pipefail; \
	${STRIP_MARKS} $< | ${NORMALIZATION_PROCEDURE} | \
	${RESTORE_LINE} > $@.tmp && mv $@.tmp $@

##################################################################
# Precision, Recall, F-Measure
test_prf:

##################################################################
# clean_test
clean_test:
	-rm -f ${EVAL_TMP_DIR}/*
