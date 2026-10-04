---
name: share-knowledge-to-memos
description: Distill shareable knowledge from the conversation into topic memos in Memos.
disable-model-invocation: true
---

# Share knowledge to Memos

**Knowledge** is a conclusion that holds for a stranger on another machine under stated conditions (tool, version, environment), backed by a basis they can check. Everything else is **state**: this machine's paths, files, hashes, installed versions, what was changed, applied, deleted, or left to do, and the session's narrative. Knowledge travels; state stays home.

The source is the current conversation. Any prompt the user adds after invoking narrows or widens that scope (a topic, a file to mine, items to skip); follow it.

## Steps

1. **Harvest.** List every candidate conclusion in scope: findings, test results, claims from docs read, gotchas hit. Done when every finding and test in scope has been considered.
2. **Distill.** Rewrite each candidate as a knowledge point, or drop it:
   - Turn state into a condition: "本机的 LoRA 是 v1.0" disappears; "v1.0 下 X 成立" survives if X is the point.
   - Give it a basis:
     - `官方｜<URL>`: the URL was opened in this session and states the claim.
     - `实测｜<方法、样本量>`: a test in this session produced it.
     - A point with neither basis is dropped.

   Done when every surviving point has a conclusion, 条件, and 依据, and a reader with zero session context understands it.
3. **Group.** Bundle points into **topic memos**: one subject a reader would search for as a unit (a model, a tool, a failure mode). A point joins the topic it is *about*; a topic with a single thin point folds into its nearest neighbour.
4. **Dedupe.** Fetch every memo matching `"知识" in tags` (follow `nextPageToken` to the end). For each topic, find any memo on the same subject. When one exists, the draft becomes that memo's merged full content: its points kept, new points added, contradicted points replaced and flagged.
5. **Review.** Show each topic draft in chat with full memo content, marked 新建 or 更新 `<memo name>`. Wait for the user to pick and edit. Done when the user names what to write.
6. **Write.** Per approved topic, exactly one call: `memo_update_memo` (`updateMask: "content"`) for 更新, `memo_create_memo` with `visibility: PRIVATE` for 新建. A failed call is reported and left; the rest proceed.
7. **Report** the memo names created and updated, and any failures.

## Memo format

```
# <主题标题>
#知识 #AI-Generated #<tag> #<tag> ...

<一句话概括这个主题最重要的结论>

实测环境：<硬件、系统、软件版本；只有含实测的 memo 写这一行>

- **<结论>**
  - 条件：<适用的工具、版本、场景>
  - 依据：官方｜<URL> 或 实测｜<方法、样本量>
```

- `#知识 #AI-Generated` lead every memo this skill writes, including updates of existing memos: `#AI-Generated` marks a memo as AI-written, apart from the user's own. Hyphenated, since Memos ends a tag at whitespace.
- Topic tags are the search surface, 4–8 per memo: tool and model names (`#PixelArtXL`, `#ComfyUI`), ecosystem (`#SDXL`, `#LoRA`), domain in Chinese (`#像素画`, `#AI绘画`), and the problem class (`#提示词`, `#显存`, `#故障`).
- Body in Chinese; technical terms keep their original English.
- Each bold conclusion stands alone: it is the answer, the sub-bullets are its warranty.
