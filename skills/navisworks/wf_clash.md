---
name: navisworks-wf-clash
description: Clash tests — running without blocking, grouping, and reporting to a coordinator. Requires the HMK Pilot connector for Navisworks.
---

Clash Detective may be absent from the session. If it is, say so plainly and
stop — no clash tool will work, and attempting one wastes a round.

## Running a test does NOT block

A real test takes minutes and runs on the Navisworks main thread. Start it,
take the JOB ID back immediately, then poll `get_clash_job`. Do not await it
and do not report failure while it is still running — telling the user it
failed invites them to run it again, which is the worst possible outcome.

While a run owns the main thread, every other model call queues behind it.
Polling the job is safe because it only reads a status dictionary.

## Results have a lifecycle

Each result is a PAIR of items with a status: New / Active / Reviewed /
Approved / Resolved. That status is the coordinator's workflow, not decoration
— never reset or overwrite it as a side effect of something else.

## Grouping is what makes a report usable

1.284 raw results are noise. Group them — by level, by source file pair, by
grid zone — and report the group counts first, with the biggest offenders
named. `list_group_by_options` tells you what the session supports.

> "412 çakışma: 280'i MEP.nwc ↔ STR.nwc, 94'ü MEP.nwc ↔ ARC.nwc. En yoğun
> bölge Level 2 (167)."

## Reporting one clash

Give the pair, the source files, and the distance. A viewpoint or an image
(`get_clash_viewpoint`, `get_clash_image`) is worth more than three sentences
of description — offer it when the user is looking at a specific clash.

## Before clearing results

Clearing is permanent and the results only come back by re-running a test that
takes minutes. Confirm with the count. See safety_destructive.
