# Anserini Regressions: MS MARCO Passage Ranking

**Models**: BM25 with (vanilla) doc2query expansions

This page documents regression experiments on the [MS MARCO passage ranking task](https://github.com/microsoft/MSMARCO-Passage-Ranking) with BM25 on (vanilla) doc2query (also called doc2query-base) expansions, as proposed in the following paper:

> Rodrigo Nogueira, Wei Yang, Jimmy Lin, and Kyunghyun Cho. [Document Expansion by Query Prediction.](https://arxiv.org/abs/1904.08375) arXiv:1904.08375, 2019.

These experiments are integrated into Anserini's regression testing framework.
For more complete instructions on how to run end-to-end experiments, refer to [this page](../../docs/experiments-doc2query.md).

The exact configurations for these regressions are stored in [this YAML file](../../src/main/resources/regression/msmarco-v1-passage.doc2query.yaml).
Note that this page is automatically generated from [this template](../../src/main/resources/docgen/templates/msmarco-v1-passage.doc2query.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```
python src/main/python/run_regression.py --index --verify --search --regression msmarco-v1-passage.doc2query
```

## Indexing

Typical indexing command:

```
bin/run.sh io.anserini.index.IndexCollection \
  -collection JsonCollection \
  -input /path/to/msmarco-passage-doc2query \
  -generator DefaultLuceneDocumentGenerator \
  -index indexes/lucene-inverted.msmarco-v1-passage.doc2query/ \
  -threads 9 -storePositions -storeDocvectors -storeRaw \
  >& logs/log.msmarco-passage-doc2query &
```

The directory `/path/to/msmarco-passage-doc2query` should be a directory containing `jsonl` files containing the expanded passage collection.
[This page](../../docs/experiments-doc2query.md) explains how to perform this data preparation.

For additional details, see explanation of [common indexing options](../../docs/common-indexing-options.md).

## Retrieval

Topics and qrels are stored [here](https://github.com/castorini/anserini-tools/tree/master/topics-and-qrels), which is linked to the Anserini repo as a submodule.
The regression experiments here evaluate on the 6980 dev set questions; see [this page](../../docs/experiments-msmarco-passage.md) for more details.

After indexing has completed, you should be able to perform retrieval as follows:

```
bin/run.sh io.anserini.search.SearchCollection \
  -index indexes/lucene-inverted.msmarco-v1-passage.doc2query/ \
  -topics tools/topics-and-qrels/topics.msmarco-passage.dev-subset.txt \
  -topicReader TsvInt \
  -output runs/run.msmarco-passage-doc2query.bm25-default.topics.msmarco-passage.dev-subset.txt \
  -bm25 &

bin/run.sh io.anserini.search.SearchCollection \
  -index indexes/lucene-inverted.msmarco-v1-passage.doc2query/ \
  -topics tools/topics-and-qrels/topics.msmarco-passage.dev-subset.txt \
  -topicReader TsvInt \
  -output runs/run.msmarco-passage-doc2query.bm25-tuned.topics.msmarco-passage.dev-subset.txt \
  -bm25 -bm25.k1 0.82 -bm25.b 0.68 &
```

Evaluation can be performed using `trec_eval`:

```
bin/trec_eval -c -m map tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-doc2query.bm25-default.topics.msmarco-passage.dev-subset.txt
bin/trec_eval -c -M 10 -m recip_rank tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-doc2query.bm25-default.topics.msmarco-passage.dev-subset.txt
bin/trec_eval -c -m recall.100 tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-doc2query.bm25-default.topics.msmarco-passage.dev-subset.txt
bin/trec_eval -c -m recall.1000 tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-doc2query.bm25-default.topics.msmarco-passage.dev-subset.txt

bin/trec_eval -c -m map tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-doc2query.bm25-tuned.topics.msmarco-passage.dev-subset.txt
bin/trec_eval -c -M 10 -m recip_rank tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-doc2query.bm25-tuned.topics.msmarco-passage.dev-subset.txt
bin/trec_eval -c -m recall.100 tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-doc2query.bm25-tuned.topics.msmarco-passage.dev-subset.txt
bin/trec_eval -c -m recall.1000 tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-doc2query.bm25-tuned.topics.msmarco-passage.dev-subset.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **AP@1000**                                                                                                  | **BM25 (default)**| **BM25 (tuned)**|
|:-------------------------------------------------------------------------------------------------------------|-----------|-----------|
| [MS MARCO Passage: Dev](https://github.com/microsoft/MSMARCO-Passage-Ranking)                                | 0.2270    | 0.2293    |
| **RR@10**                                                                                                    | **BM25 (default)**| **BM25 (tuned)**|
| [MS MARCO Passage: Dev](https://github.com/microsoft/MSMARCO-Passage-Ranking)                                | 0.2189    | 0.2213    |
| **R@100**                                                                                                    | **BM25 (default)**| **BM25 (tuned)**|
| [MS MARCO Passage: Dev](https://github.com/microsoft/MSMARCO-Passage-Ranking)                                | 0.7133    | 0.7171    |
| **R@1000**                                                                                                   | **BM25 (default)**| **BM25 (tuned)**|
| [MS MARCO Passage: Dev](https://github.com/microsoft/MSMARCO-Passage-Ranking)                                | 0.8900    | 0.8911    |

Explanation of settings:

+ The setting "default" refers the default BM25 settings of `k1=0.9`, `b=0.4`.
+ The setting "tuned" refers to `k1=0.82`, `b=0.68`, tuned on _on the original passages_, as described in [this page](../../docs/experiments-msmarco-passage.md).

## Additional Implementation Details

Note that prior to December 2021, runs generated with `SearchCollection` in the TREC format and then converted into the MS MARCO format give slightly different results from runs generated by `SearchMsmarco` directly in the MS MARCO format, due to tie-breaking effects.
This was fixed with [#1458](https://github.com/castorini/anserini/issues/1458), which also introduced (intra-configuration) multi-threading.
As a result, `SearchMsmarco` has been deprecated and replaced by `SearchCollection`; both have been verified to generate _identical_ output.

The commands below have been retained for historical reasons only.

The following command generates with `SearchMsmarco` the run denoted "BM25 (tuned)" above (`k1=0.82`, `b=0.68`):

```bash
$ sh target/appassembler/bin/SearchMsmarco -hits 1000 -threads 8 \
    -index indexes/lucene-index.msmarco-passage-doc2query.pos+docvectors+raw \
    -queries collections/msmarco-passage/queries.dev.small.tsv \
    -k1 0.82 -b 0.68 \
    -output runs/run.msmarco-passage-doc2query

$ python tools/scripts/msmarco/msmarco_passage_eval.py \
   collections/msmarco-passage/qrels.dev.small.tsv runs/run.msmarco-passage-doc2query

#####################
MRR @10: 0.2213412471005586
QueriesRanked: 6980
#####################
```

Note that this run does _not_ correspond to the scores reported in the paper that introduced doc2query:

> Rodrigo Nogueira, Wei Yang, Jimmy Lin, and Kyunghyun Cho. [Document Expansion by Query Prediction.](https://arxiv.org/abs/1904.08375) arXiv:1904.08375, 2019.

The scores reported in the above paper refer to entry "BM25 (Anserini) + doc2query" dated 2019/04/10 on the [MS MARCO Passage Ranking Leaderboard](https://microsoft.github.io/msmarco/).
The paper/leaderboard run reports 0.215 MRR@10, which is slightly lower than the "BM25 (Tuned)" regression run above, due to an earlier version of Lucene (7.6) and use of default BM25 parameters.