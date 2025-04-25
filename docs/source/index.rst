Welcome to GraphGen's documentation!
===================================

GraphGen is a framework for synthetic data generation guided by knowledge graphs. Here is our `paper <https://github.com/open-sciencelab/GraphGen/tree/main/resources/GraphGen.pdf>`_.

It begins by constructing a fine-grained knowledge graph from the source text, then identifies knowledge gaps in LLMs using the expected calibration error metric, prioritizing the generation of QA pairs that target high-value, long-tail knowledge.

Furthermore, GraphGen incorporates multi-hop neighborhood sampling to capture complex relational information and employs style-controlled generation to diversify the resulting QA data.

.. note::

   This project is under active development.

Contents
--------

.. toctree::

   quick_start
   parameters
