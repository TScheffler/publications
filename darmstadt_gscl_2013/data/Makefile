##################################################################
# Variables
NORMALIZED_DIR   := normalized
UNNORMALIZED_DIR := unnormalized

##################################################################
# Targets
.PHONY: all noise_cleaner_test \
	clean clean_noise_cleaner_test

##################################################################
# all
all: noise_cleaner_test

clean: clean_noise_cleaner_test

##################################################################
# noise_cleaner_test
noise_cleaner_test: ${NORMALIZED_DIR}/1000_tweets_noise_normalization.res

${NORMALIZED_DIR}/1000_tweets_noise_normalization.res: %.res: %.in %.out
	set -e -o pipefail; \
	paste $^ | awk 'NF{sub(/\t/, "\n"); print $0 "\n"}' > $@.tmp; \
	mv $@.tmp $@

${NORMALIZED_DIR}/1000_tweets_noise_normalization.in: 10000_random_tweets.txt
	./select_random_lines -n 1000 $< > $@.tmp; \
	mv $@.tmp $@

${NORMALIZED_DIR}/1000_tweets_noise_normalization.out: %.out : %.in
	noise_cleaner $< > $@.tmp; \
	mv $@.tmp $@

clean_noise_cleaner_test:
	-rm -rf $(wildcard $(NORMALIZED_DIR)/1000_tweets_noise_normalization*)
