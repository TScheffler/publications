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
