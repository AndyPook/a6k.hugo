---
title: LuceneNet Sparse Faceted Search
date: 2012-12-10T15:44:00Z
summary: Faceted Search for large cardinality
tags: [dotnet, lucene]
---

I recently had a use for [faceted search](http://en.wikipedia.org/wiki/Faceted_search) in a project using Lucene.net. The contrib project for [Lucene.Net](http://lucenenet.apache.org/) provides SimpleFacetedSearch which is great, but… it becomes very inefficient when your index has lots of values in the given field. So, for example, if you have a product category code that only has a hand full of values then SimpleFS is fine. If you facet against date and you have significant history then SimpleFS will eat all your memory very quickly.

SimpleFS represents a facet as a bitmap for each value. The size of the bitmap is equal to the number of documents in your index in bits (so numDocs/8 bytes). A quick calculation based on your index will give you very big numbers if you have large numbers of documents, values or both. In our case we have 100K values and 3.5M documents = 100K * (3.5M/8) = around 41GB!

[SparseFacetedSearch](https://github.com/Artesian/SparseFacetedSearch) to the rescue. SparseFS can deal with facets with many thousands of values (100’s of thousands in my case). Using much less memory. It is based on SimpleFacetedSearch but uses DocID lists instead of bitmaps. It’s suitable for high cardinality, sparsely populated facets. i.e. There are a large number of facet values and each facet value is hit in a small percentage of documents.
The memory usage is related to the number of hits each document has. In the product category example this would be exactly 1. The break even point is if you have more than 32 values. In our case we generally have between 1 and 5 values per document. Our memory usage is around 600MB. Quite a saving on 41GB!

There is a [bunch of math](https://github.com/Artesian/SparseFacetedSearch/blob/master/README.md) that goes to explaining how that works out.