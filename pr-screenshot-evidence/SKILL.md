---
name: pr-screenshot-evidence
description: Produce and attach complete screenshot evidence to a pull request. Use when a pull request changes a visible screen, a permission, or other user-facing behavior, so reviewers see every case the change touches. Skip when the change has no user-visible surface.
---

# PR screenshot evidence

A pull request that changes what users see or what they are allowed to do should
ship with screenshots that prove it. The failure mode this skill prevents is
partial evidence: one happy-path screenshot that hides the denied state, the
empty state, or the mobile layout.

## When to use

Use this skill when a pull request changes a visible screen, a permission, or
user-facing behavior. If it does not, state it plainly and move on:

> no user-visible change - screenshots not applicable

## Step 1 - Build the case matrix (mandatory)

Before capturing anything, list every case the change touches in a table. One
row per case. Include:

- every role or grant involved in the change;
- the allowed, denied, empty, error, and loading states where relevant;
- desktop, mobile, light, and dark only where the change touches them;
- base versus head whenever behavior changed between them.

Example matrix (neutral placeholders):

| #  | Case                          | Surface            | Variant     |
|----|-------------------------------|--------------------|-------------|
| 1  | `admin` sees settings         | desktop, head      | allowed     |
| 2  | `viewer` sees denied message  | desktop, head      | denied      |
| 3  | new user sees empty state     | desktop, head      | empty       |
| 4  | same as 1, before the change  | desktop, base      | allowed     |
| 5  | same as 2, before the change  | desktop, base      | denied      |
| 6  | same as 1 on a narrow screen  | mobile, head       | allowed     |

Capture every row in matrix order, and give every row a one-line caption
stating what its image proves. A case that cannot be captured (no fixture
available, state not reachable) stays in the matrix marked `NOT CAPTURED` with
the reason. It is never silently dropped.

## Step 2 - Capture

- Fix one viewport size and one theme per set of captures; change them only
  when the matrix row says to.
- Use disposable, synthetic fixture data only. Never screenshot real tenant,
  customer, or user data, and never capture internal hostnames or credentials.
- Start the services you need, capture, then stop them cleanly.
- Leave the working tree clean: never commit image files to the repository.
- Give every image a filename derived from its matrix number, such as
  `case-01-allowed.png`.

## Step 3 - Write the manifest

Keep one row per image, next to the images but outside the repository
(`manifest-template.csv` from this skill is a starting point):

- filename
- case (the matrix row)
- caption
- sha256
- the head commit the image was captured from

Verify every hash before attaching anything: build a checksums file from the
manifest's filename and sha256 columns, then check it with
`shasum -a 256 -c` (macOS/BSD) or `sha256sum -c` (Linux), for example:

```sh
awk -F, '{print $4 "  " $1}' manifest.csv > checksums.txt
shasum -a 256 -c checksums.txt   # or: sha256sum -c checksums.txt
```

A hash that does not match means the image is stale; recapture it.

## Step 4 - Attach to the pull request

Images go into the pull request description under a `## Screenshots` heading by
default. Use a comment only when asked. Put them in matrix order, each image
followed or preceded by its one-line caption stating what it proves. After the
images, list the `NOT CAPTURED` cases with their reasons, plus any limits
(slow loads, flaky states).

Images are never committed to the repository; they live only in the
description. Routes, in order of preference:

1. **Agent uploads.** Drive a signed-in browser to the description editor and
   drop each file onto the editor. Upload in small batches (large batches can
   hit an upload limit). Verify every upload succeeded and the count matches
   the manifest before saving. Never save a description unless every step
   succeeded.
2. **Human uploads.** When the agent's browser is not signed in, ask a human to
   attach the files through the editor's paperclip control, using the manifest
   as the checklist.
3. **Local delivery.** When uploading is not possible, save the images and the
   manifest locally and report the path so a human can attach them.

Never save a half-built description over the existing one: keep the previous
text and be ready to restore it from the edit history. After saving, check the
rendered description: every image must show as a rendered attachment, and a
literal `Failed to upload` marker in the text means that image did not make it.

## Step 5 - Clean up

Copy anything still needed (manifest, images, notes) out of the working copy
before it is removed. The repository keeps only text.
