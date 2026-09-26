Please double-check this from our actual installed Llama Cloud SDK and implementation. Do not assume the feature is unavailable just because our current code does not use it.

I specifically want to determine whether `llama-cloud==2.13.0` supports granular bounding boxes at:

- table cell level
- word level
- line level

Public LlamaParse documentation indicates that version 2.13.0 has an `OutputOptions` field similar to:

```python
granular_bboxes: List[Literal["cell", "line", "word"]]
```

and that `"cell"` means one bounding box for each detected table cell.

Please verify this locally rather than relying only on general documentation.

1. Confirm the exact installed versions of `llama-cloud`, `llama-parse`, or any related LlamaParse packages we use.

2. Inspect the installed `llama_cloud` package itself, especially:

```text
llama_cloud/types/parsing_create_params.py
```

Search for:

```text
granular_bboxes
grounded_items
word_bbox
```

3. Check whether `OutputOptions` in OUR installed version contains:

```python
granular_bboxes
```

and show me the exact definition and comments from the installed package.

4. Check our actual LlamaParse request code. Tell me whether we currently send something like:

```python
output_options={
    "granular_bboxes": ["cell", "word"]
}
```

If we do not send it, clearly separate:

- "our application currently doesn't request it"

from

- "the SDK/API doesn't support it"

Those are different conclusions.

5. Check how the results are returned. My understanding is that granular boxes may NOT appear directly in the normal parsed items. Instead, when requested, they may be written to a separate `grounded_items` JSONL output and referenced through the result metadata.

Verify whether this is true in our installed SDK.

6. Also verify this separate option:

```python
additional_outputs=["word_bbox"]
```

and explain how that differs from:

```python
granular_bboxes=["word"]
```

7. If possible, using an existing safe test PDF in our development environment, run a very small test containing a table and request:

```python
output_options={
    "granular_bboxes": ["cell", "word"]
}
```

Do not modify production code.

Then inspect the COMPLETE response, including metadata and any linked/side output. I specifically want to know whether a table with, for example:

| Name | Amount |
| ABC | $12,500 |

produces:

- one bounding box for the whole table
- separate bounding boxes for the `Name`, `Amount`, `ABC`, and `$12,500` cells
- word-level boxes inside those cells

8. If the request fails, capture the exact error. Determine whether the reason is:

- our installed SDK
- our LlamaParse server/API version
- the parsing tier/version we use
- our account/configuration
- incorrect request syntax
- or the feature simply not being enabled in our current code

Do not just answer yes/no.

Give me a short final answer in this format:

**SDK supports it:** Yes / No  
**Our backend supports it:** Yes / No / Not proven  
**Our current code enables it:** Yes / No  
**Cell-level table boxes:** Yes / No / Not proven  
**Word-level boxes:** Yes / No / Not proven  
**Where the boxes are returned:**  
**What needs to change in our code:**  
**Evidence:** exact file names, fields, response fields, or errors you found.

Use simple language and show evidence for each conclusion.
