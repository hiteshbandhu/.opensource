<div align="center">

<img src="https://github.com/hiteshbandhu.png" width="150" height="150" alt="Hitesh Bandhu" style="border-radius: 50%; border: 3px solid #0969da;"/>

# Hitesh Bandhu

**Open Source Contributor** • Chandigarh, India

i manipulate language models, build and experiment with them

[GitHub](https://github.com/hiteshbandhu) • [Web Portfolio](https://hiteshbandhu.github.io/.opensource/) • [Twitter](https://twitter.com/_hiteshbandhu)

---

### Portfolio Summary

</div>

<table align="center">
  <tr>
    <td align="center" width="200" style="padding: 20px;">
      <h2 style="margin: 0; font-size: 2.5em;">1</h2>
      <p style="margin: 5px 0 0 0; font-weight: 600;">Contributions</p>
    </td>
    <td align="center" width="200" style="padding: 20px;">
      <h2 style="margin: 0; font-size: 2.5em;">1</h2>
      <p style="margin: 5px 0 0 0; font-weight: 600;">Projects</p>
    </td>
    <td align="center" width="200" style="padding: 20px;">
      <h2 style="margin: 0; font-size: 2.5em;">1</h2>
      <p style="margin: 5px 0 0 0; font-weight: 600;">Languages</p>
    </td>
  </tr>
</table>

<br/>

---

## Contributions by Project


### <img src="https://github.com/CapSoftware.png" width="28" height="28" align="absmiddle" alt="CapSoftware"> [CapSoftware/Cap](https://github.com/CapSoftware/Cap)

Open source Loom alternative. Beautiful, shareable screen recordings.

**1 contribution** • Rust • **★ 23,368**

---

#### <img src="https://raw.githubusercontent.com/primer/octicons/main/icons/git-merge-16.svg" width="16" height="16" align="absmiddle" alt="Merged" title="Merged pull request"> [skip still re-renders for annotation-only screenshot edits (−95–98% CPU while annotating)](https://github.com/CapSoftware/Cap/pull/2543) <sub>[#2543](https://github.com/CapSoftware/Cap/pull/2543)</sub>

<sub><strong>Merged</strong> on Oct 14, 2026</sub>

- **Changes:** <code>+332</code> <code>-0</code> across 1 file
- **Language:** Rust
- **Merged by:** [@richiemcilroy](https://github.com/richiemcilroy)


<details>
<summary>View PR description</summary>

**What it fixes**

Every annotation edit in the screenshot editor sent the whole `ProjectConfiguration` to `run_still_renderer`. That includes each tick of the stroke width, opacity and mask level sliders, every colour change, and every committed stroke, move, resize, paste or delete. Each send triggered a full render of the still on the GPU, a readback of the padded frame, `frame_buffers` (unpad, clone and swap to BGRA), a new `RenderImage` for gpui to upload, and `drop_image` on the old one.

The still renderer never reads `annotations`. They are painted as gpui overlays in the editor and composited by `screenshot_export::composite` on export. So all of that work produced a frame that was byte-for-byte the frame already on screen.

`publish()` now goes through a small `RenderInputGate`. It keys the config with `annotations` taken out (plus `hidden_text_segments`, which is `skip_serializing`) and only sends when that key changes. Padding, crop, background, border, shadow and undo/redo of those still render exactly as before. Annotation edits still record history, save, and refresh the mask overlays; they no longer kick the GPU.

Users feel this as slider drags and annotation edits that no longer stutter on large Retina or 4K screenshots. It also takes the renderer's ~33 MB-per-tick readback and upload churn off the machine while annotating.

**Benchmark** (120-tick stroke-width slider drag at 60 Hz through the real `run_still_renderer` and the editor's `frame_buffers` pump; process CPU from `getrusage` over the drag plus a 750 ms drain; median of 3 runs on a GitHub `macos-latest` runner, Apple M1 (Virtual), Metal)

| still | renders (before → after) | process CPU over the drag | CPU per tick | median edit-to-frame before |
| --- | ---: | ---: | ---: | ---: |
| 1920×1080 | 119 → 0 | 735 ms → 36 ms (−95%) | 6.13 → 0.30 ms | 11.5 ms |
| 3024×1964 | 120 → 0 | 1655 ms → 39 ms (−98%) | 13.79 → 0.32 ms | 20.3 ms |
| 3840×2160 | 118 → 0 | 1840 ms → 31 ms (−98%) | 15.33 → 0.26 ms | 23.7 ms |

After the change, what's left of the "after" CPU is the benchmark's own 60 Hz loop and tokio idling; the gate check itself is a serde serialization of the config.

GPU time and gpui's atlas upload of each new frame aren't in the CPU column, so the real saving is larger. Before the change, the renderer was busy for the whole drag (the 4K edit-to-frame time is longer than the 16.7 ms tick).

```bash
cd apps/desktop-gpui
cargo test --bin cap-gpui benchmark_annotation_slider_drag -- --ignored --nocapture
```

"Before" is the same drag with every tick published, which is what `publish()` did on `main` (f6164c1). "After" is the drag through `RenderInputGate`.

<details><summary>All three runs</summary>

| run | still | main renders / CPU | gated renders / CPU |
| --- | --- | ---: | ---: |
| 1 | 1920×1080 | 112 / 735 ms | 0 / 29 ms |
| 1 | 3024×1964 | 120 / 1694 ms | 0 / 27 ms |
| 1 | 3840×2160 | 113 / 2281 ms | 0 / 24 ms |
| 2 | 1920×1080 | 119 / 605 ms | 0 / 36 ms |
| 2 | 3024×1964 | 120 / 1194 ms | 0 / 42 ms |
| 2 | 3840×2160 | 118 / 1840 ms | 0 / 31 ms |
| 3 | 1920×1080 | 120 / 844 ms | 0 / 37 ms |
| 3 | 3024×1964 | 117 / 1655 ms | 0 / 39 ms |
| 3 | 3840×2160 | 119 / 1711 ms | 0 / 38 ms |

&lt;/details&gt;

**Checks**

- New unit tests:
  - annotation-only edits (stroke width, colour, adding and clearing annotations) never pass the gate;
  - padding, rounding, background and `hidden_text_segments` changes always do, even when combined with an annotation edit;
  - annotations stay on the project after the check;
  - a freshly loaded editor sends its first config.
- `cargo test --bin cap-gpui`: 1081 passed. `cargo test -p cap-gpui-kernels`: 22 passed.
- `cargo fmt --check` is clean.
- `cargo clippy --bin cap-gpui --all-targets -- -D warnings` (1.95.0) reports nothing in `screenshot_editor.rs`. The errors it does report are pre-existing ones in files this PR doesn't touch (`editor_clips.rs`, `editor_crop.rs`, `editor_edits.rs`, `editor_window.rs`, `editor_timeline.rs`, `editor_sidebar/animated_gradient.rs`, `updates.rs`, `diagnostics.rs`).
- `screenshot_export` still gets the full config, annotations included, through `ExportRequest::Render`, so copy and save output is unchanged.








<h2><a href="https://app.greptile.com/api/retrigger?id=80391512"><picture><source media="(prefers-color-scheme: dark)" srcset="https://greptile-static-assets.s3.amazonaws.com/badges/RetriggerDark.svg?v=2"><source media="(prefers-color-scheme: light)" srcset="https://greptile-static-assets.s3.amazonaws.com/badges/Retrigger.svg?v=2"><img alt="Retrigger" src="https://greptile-static-assets.s3.amazonaws.com/badges/Retrigger.svg?v=2" align="right"></picture></a>Confidence Score: 5/5</h2>

 The PR appears safe to merge, with the Windows test-build issue addressed.

<h3>Summary</h3>

The PR skips still-image rendering when only annotations change. It adds gate tests and an ignored GPU benchmark.

- Annotation edits no longer send duplicate frames to the still renderer.



<sub>Reviews (2) · Last reviewed commit: ["test(desktop-gpui): read benchmark CPU t..."](https://github.com/capsoftware/cap/commit/3d99e55da39b8376eabeb16d9f1a17261dccc5b8) · [Reviewed by Greptile](https://www.greptile.com/?utm_source=greptile_expert&utm_medium=github&utm_campaign=code_reviews&utm_content=capsoftware/cap)</sub>

</details>



---



---

<div align="center">

<sub>Last updated: October 16, 2026</sub>

<br/>

<sub>Generated by <a href="https://github.com/opensource-showcase/opensource-showcase">opensource-showcase</a> • View raw data: <a href="contributions.json">contributions.json</a></sub>

</div>
