---
title: SpanRuler
tag: class
source: spacy/pipeline/span_ruler.py
new: 3.3.1
teaser: 'Pipeline component for rule-based span and named entity recognition'
api_string_name: span_ruler
api_trainable: false
---

The span ruler lets you add spans to [`Doc.spans`](/api/doc#spans) and/or
[`Doc.ents`](/api/doc#ents) using token-based rules or exact phrase matches. For
usage examples, see the docs on
[rule-based span matching](/usage/rule-based-matching#spanruler).

## Assigned Attributes {#assigned-attributes}

Matches will be saved to `Doc.spans[spans_key]` as a
[`SpanGroup`](/api/spangroup) and/or to `Doc.ents`, where the annotation is
saved in the `Token.ent_type` and `Token.ent_iob` fields.

| Location               | Value                                                             |
| ---------------------- | ----------------------------------------------------------------- |
| `Doc.spans[spans_key]` | The annotated spans. ~~SpanGroup~~                                |
| `Doc.ents`             | The annotated spans. ~~Tuple[Span]~~                              |
| `Token.ent_iob`        | An enum encoding of the IOB part of the named entity tag. ~~int~~ |
| `Token.ent_iob_`       | The IOB part of the named entity tag. ~~str~~                     |
| `Token.ent_type`       | The label part of the named entity tag (hash). ~~int~~            |
| `Token.ent_type_`      | The label part of the named entity tag. ~~str~~                   |

## Config and implementation {#config}

The default config is defined by the pipeline component factory and describes
how the component should be configured. You can override its settings via the
`config` argument on [`nlp.add_pipe`](/api/language#add_pipe) or in your
[`config.cfg`](/usage/training#config).

> #### Example
>
> ```python
> config = {
>    "spans_key": "my_spans",
>    "validate": True,
>    "overwrite": False,
> }
> nlp.add_pipe("span_ruler", config=config)
> ```

| Setting               | Description                                                                                                                                                                             |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `spans_key`           | The spans key to save the spans under. If `None`, no spans are saved. Defaults to `"ruler"`. ~~Optional[str]~~                                                                          |
| `spans_filter`        | The optional method to filter spans before they are assigned to doc.spans. Defaults to `None`. ~~Optional[Callable[[Iterable[Span], Iterable[Span]], List[Span]]]~~                     |
| `annotate_ents`       | Whether to save spans to doc.ents. Defaults to `False`. ~~bool~~                                                                                                                        |
| `ents_filter`         | The method to filter spans before they are assigned to doc.ents. Defaults to `util.filter_chain_spans`. ~~Callable[[Iterable[Span], Iterable[Span]], List[Span]]~~                      |
| `phrase_matcher_attr` | Token attribute to match on, passed to the internal PhraseMatcher as `attr`. Defaults to `None`. ~~Optional[Union[int, str]]~~                                                          |
| `validate`            | Whether patterns should be validated, passed to Matcher and PhraseMatcher as `validate`. Defaults to `False`. ~~bool~~                                                                  |
| `overwrite`           | Whether to remove any existing spans under `Doc.spans[spans key]` if `spans_key` is set, or to remove any ents under `Doc.ents` if `annotate_ents` is set. Defaults to `True`. ~~bool~~ |
| `scorer`              | The scoring method. Defaults to [`Scorer.score_spans`](/api/scorer#score_spans) for `Doc.spans[spans_key]` with overlapping spans allowed. ~~Optional[Callable]~~                       |

```python
%%GITHUB_SPACY/spacy/pipeline/span_ruler.py
```

## SpanRuler.\_\_init\_\_ {#init tag="method"}

Initialize the span ruler. If patterns are supplied here, they need to be a list
of dictionaries with a `"label"` and `"pattern"` key. A pattern can either be a
token pattern (list) or a phrase pattern (string). For example:
`{"label": "ORG", "pattern": "Apple"}`.

> #### Example
>
> ```python
> # Construction via add_pipe
> ruler = nlp.add_pipe("span_ruler")
>
> # Construction from class
> from spacy.pipeline import SpanRuler
> ruler = SpanRuler(nlp, overwrite=True)
> ```

| Name                  | Description                                                                                                                                                                                                                         |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `nlp`                 | The shared nlp object to pass the vocab to the matchers and process phrase patterns. ~~Language~~                                                                                                                                   |
| `name`                | Instance name of the current pipeline component. Typically passed in automatically from the factory when the component is added. Used to disable the current span ruler while creating phrase patterns with the nlp object. ~~str~~ |
| _keyword-only_        |                                                                                                                                                                                                                                     |
| `spans_key`           | The spans key to save the spans under. If `None`, no spans are saved. Defaults to `"ruler"`. ~~Optional[str]~~                                                                                                                      |
| `spans_filter`        | The optional method to filter spans before they are assigned to doc.spans. Defaults to `None`. ~~Optional[Callable[[Iterable[Span], Iterable[Span]], List[Span]]]~~                                                                 |
| `annotate_ents`       | Whether to save spans to doc.ents. Defaults to `False`. ~~bool~~                                                                                                                                                                    |
| `ents_filter`         | The method to filter spans before they are assigned to doc.ents. Defaults to `util.filter_chain_spans`. ~~Callable[[Iterable[Span], Iterable[Span]], List[Span]]~~                                                                  |
| `phrase_matcher_attr` | Token attribute to match on, passed to the internal PhraseMatcher as `attr`. Defaults to `None`. ~~Optional[Union[int, str]]~~                                                                                                      |
| `validate`            | Whether patterns should be validated, passed to Matcher and PhraseMatcher as `validate`. Defaults to `False`. ~~bool~~                                                                                                              |
| `overwrite`           | Whether to remove any existing spans under `Doc.spans[spans key]` if `spans_key` is set, or to remove any ents under `Doc.ents` if `annotate_ents` is set. Defaults to `True`. ~~bool~~                                             |
| `scorer`              | The scoring method. Defaults to [`Scorer.score_spans`](/api/scorer#score_spans) for `Doc.spans[spans_key]` with overlapping spans allowed. ~~Optional[Callable]~~                                                                   |

## SpanRuler.initialize {#initialize tag="method"}

Initialize the component with data and used before training to load in rules
from a [pattern file](/usage/rule-based-matching/#spanruler-files). This method
is typically called by [`Language.initialize`](/api/language#initialize) and
lets you customize arguments it receives via the
[`[initialize.components]`](/api/data-formats#config-initialize) block in the
config. Any existing patterns are removed on initialization.

> #### Example
>
> ```python
> span_ruler = nlp.add_pipe("span_ruler")
> span_ruler.initialize(lambda: [], nlp=nlp, patterns=patterns)
> ```
>
> ```ini
> ### config.cfg
> [initialize.components.span_ruler]
>
> [initialize.components.span_ruler.patterns]
> @readers = "srsly.read_jsonl.v1"
> path = "corpus/span_ruler_patterns.jsonl
> ```

| Name           | Description                                                                                                                                                        |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `get_examples` | Function that returns gold-standard annotations in the form of [`Example`](/api/example) objects. Not used by the `SpanRuler`. ~~Callable[[], Iterable[Example]]~~ |
| _keyword-only_ |                                                                                                                                                                    |
| `nlp`          | The current `nlp` object. Defaults to `None`. ~~Optional[Language]~~                                                                                               |
| `patterns`     | The list of patterns. Defaults to `None`. ~~Optional[Sequence[Dict[str, Union[str, List[Dict[str, Any]]]]]]~~                                                      |

## SpanRuler.\_\len\_\_ {#len tag="method"}

The number of all patterns added to the span ruler.

> #### Example
>
> ```python
> ruler = nlp.add_pipe("span_ruler")
> assert len(ruler) == 0
> ruler.add_patterns([{"label": "ORG", "pattern": "Apple"}])
> assert len(ruler) == 1
> ```

| Name        | Description                     |
| ----------- | ------------------------------- |
| **RETURNS** | The number of patterns. ~~int~~ |

## SpanRuler.\_\_contains\_\_ {#contains tag="method"}

Whether a label is present in the patterns.

> #### Example
>
> ```python
> ruler = nlp.add_pipe("span_ruler")
> ruler.add_patterns([{"label": "ORG", "pattern": "Apple"}])
> assert "ORG" in ruler
> assert not "PERSON" in ruler
> ```

| Name        | Description                                         |
| ----------- | --------------------------------------------------- |
| `label`     | The label to check. ~~str~~                         |
| **RETURNS** | Whether the span ruler contains the label. ~~bool~~ |

## SpanRuler.\_\_call\_\_ {#call tag="method"}

Find matches in the `Doc` and add them to `doc.spans[span_key]` and/or
`doc.ents`. Typically, this happens automatically after the component has been
added to the pipeline using [`nlp.add_pipe`](/api/language#add_pipe). If the
span ruler was initialized with `overwrite=True`, existing spans and entities
will be removed.

> #### Example
>
> ```python
> ruler = nlp.add_pipe("span_ruler")
> ruler.add_patterns([{"label": "ORG", "pattern": "Apple"}])
>
> doc = nlp("A text about Apple.")
> spans = [(span.text, span.label_) for span in doc.spans["ruler"]]
> assert spans == [("Apple", "ORG")]
> ```

| Name        | Description                                                          |
| ----------- | -------------------------------------------------------------------- |
| `doc`       | The `Doc` object to process, e.g. the `Doc` in the pipeline. ~~Doc~~ |
| **RETURNS** | The modified `Doc` with added spans/entities. ~~Doc~~                |

## SpanRuler.add_patterns {#add_patterns tag="method"}

Add patterns to the span ruler. A pattern can either be a token pattern (list of
dicts) or a phrase pattern (string). For more details, see the usage guide on
[rule-based matching](/usage/rule-based-matching).

> #### Example
>
> ```python
> patterns = [
>     {"label": "ORG", "pattern": "Apple"},
>     {"label": "GPE", "pattern": [{"lower": "san"}, {"lower": "francisco"}]}
> ]
> ruler = nlp.add_pipe("span_ruler")
> ruler.add_patterns(patterns)
> ```

| Name       | Description                                                      |
| ---------- | ---------------------------------------------------------------- |
| `patterns` | The patterns to add. ~~List[Dict[str, Union[str, List[dict]]]]~~ |

## SpanRuler.remove {#remove tag="method"}

Remove patterns by label from the span ruler. A `ValueError` is raised if the
label does not exist in any patterns.

> #### Example
>
> ```python
> patterns = [{"label": "ORG", "pattern": "Apple", "id": "apple"}]
> ruler = nlp.add_pipe("span_ruler")
> ruler.add_patterns(patterns)
> ruler.remove("ORG")
> ```

| Name    | Description                            |
| ------- | -------------------------------------- |
| `label` | The label of the pattern rule. ~~str~~ |

## SpanRuler.remove_by_id {#remove_by_id tag="method"}

Remove patterns by ID from the span ruler. A `ValueError` is raised if the ID
does not exist in any patterns.

> #### Example
>
> ```python
> patterns = [{"label": "ORG", "pattern": "Apple", "id": "apple"}]
> ruler = nlp.add_pipe("span_ruler")
> ruler.add_patterns(patterns)
> ruler.remove_by_id("apple")
> ```

| Name         | Description                         |
| ------------ | ----------------------------------- |
| `pattern_id` | The ID of the pattern rule. ~~str~~ |

## SpanRuler.clear {#clear tag="method"}

Remove all patterns the span ruler.

> #### Example
>
> ```python
> patterns = [{"label": "ORG", "pattern": "Apple", "id": "apple"}]
> ruler = nlp.add_pipe("span_ruler")
> ruler.add_patterns(patterns)
> ruler.clear()
> ```

## SpanRuler.to_disk {#to_disk tag="method"}

Save the span ruler patterns to a directory. The patterns will be saved as
newline-delimited JSON (JSONL).

> #### Example
>
> ```python
> ruler = nlp.add_pipe("span_ruler")
> ruler.to_disk("/path/to/span_ruler")
> ```

| Name   | Description                                                                                                                                |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `path` | A path to a directory, which will be created if it doesn't exist. Paths may be either strings or `Path`-like objects. ~~Union[str, Path]~~ |

## SpanRuler.from_disk {#from_disk tag="method"}

Load the span ruler from a path.

> #### Example
>
> ```python
> ruler = nlp.add_pipe("span_ruler")
> ruler.from_disk("/path/to/span_ruler")
> ```

| Name        | Description                                                                                     |
| ----------- | ----------------------------------------------------------------------------------------------- |
| `path`      | A path to a directory. Paths may be either strings or `Path`-like objects. ~~Union[str, Path]~~ |
| **RETURNS** | The modified `SpanRuler` object. ~~SpanRuler~~                                                  |

## SpanRuler.to_bytes {#to_bytes tag="method"}

Serialize the span ruler to a bytestring.

> #### Example
>
> ```python
> ruler = nlp.add_pipe("span_ruler")
> ruler_bytes = ruler.to_bytes()
> ```

| Name        | Description                        |
| ----------- | ---------------------------------- |
| **RETURNS** | The serialized patterns. ~~bytes~~ |

## SpanRuler.from_bytes {#from_bytes tag="method"}

Load the pipe from a bytestring. Modifies the object in place and returns it.

> #### Example
>
> ```python
> ruler_bytes = ruler.to_bytes()
> ruler = nlp.add_pipe("span_ruler")
> ruler.from_bytes(ruler_bytes)
> ```

| Name         | Description                                    |
| ------------ | ---------------------------------------------- |
| `bytes_data` | The bytestring to load. ~~bytes~~              |
| **RETURNS**  | The modified `SpanRuler` object. ~~SpanRuler~~ |

## SpanRuler.labels {#labels tag="property"}

All labels present in the match patterns.

| Name        | Description                            |
| ----------- | -------------------------------------- |
| **RETURNS** | The string labels. ~~Tuple[str, ...]~~ |

## SpanRuler.ids {#ids tag="property"}

All IDs present in the `id` property of the match patterns.

| Name        | Description                         |
| ----------- | ----------------------------------- |
| **RETURNS** | The string IDs. ~~Tuple[str, ...]~~ |

## SpanRuler.patterns {#patterns tag="property"}

All patterns that were added to the span ruler.

| Name        | Description                                                                              |
| ----------- | ---------------------------------------------------------------------------------------- |
| **RETURNS** | The original patterns, one dictionary per pattern. ~~List[Dict[str, Union[str, dict]]]~~ |

## Attributes {#attributes}

| Name             | Description                                                                      |
| ---------------- | -------------------------------------------------------------------------------- |
| `key`            | The spans key that spans are saved under. ~~Optional[str]~~                      |
| `matcher`        | The underlying matcher used to process token patterns. ~~Matcher~~               |
| `phrase_matcher` | The underlying phrase matcher used to process phrase patterns. ~~PhraseMatcher~~ |
