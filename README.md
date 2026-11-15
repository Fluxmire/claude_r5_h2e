# Image to Three.js miner (claude_r5 H2e)

Entry point: `main.py` -> `async generate(image, seed, witness) -> str | None` (the competition witness runtime).
The active plan is `ACTIVE_PLAN` in `miner404/versions.py`.

## Base and changes

Base: [SomPeer/copperbellows@d481cfc](https://github.com/SomPeer/copperbellows/tree/d481cfc14057e25d5618a76098ecc54343639f30)
(its code is identical to the round-4 winner TiltGilt/VertexLoom@2385ec1; only the plan differs).

Plan relative to the round-4 winner's:

- same 13 gated rewrite steps, fallbacks, `judge_pick` gates, `replan`, `judged_tilt`, `outline_fit_last`, and
  SEARCH/REPLACE patch edits on rewrite steps 8-13;
- 6 Sol-high drafts as 2 plain + 2 proportions + 2 inventory (parts, counts, colours listed before building);
- no draft merge, no tone pass;
- `routed=True`: one Sol-low call (request 9995) labels the photo; upright furniture/structures skip both the photo
  tilt and the judged tilt, tall photos skip the photo tilt. A refused or failed label call leaves the base behaviour.
- finishing after the judged tilt, each step kept only if the renderer accepts it:
  - `texfix`: data textures left at three.js's NearestFilter defaults get trilinear sampling with mipmaps;
  - judged stretch: the module's shorter bounding-box axes are enlarged (never past 0.999 of the cube; round bodies keep
    a round section; modules naming a spout are left alone). x1.25 is used unless x1.0, x1.15 or x1.4 beats it in the
    selector (requests 9980/9981, both orders) by `STRETCH_MARGIN`; a refused call keeps x1.25.
  - toward the cameras (last step): the finished object slides along +z until its box front reaches 0.4995, so it
    reads larger in the judge's front views while staying inside the unit cube.

Code changes: `miner404/route.py` (new), `miner404/tilt.py` (`photo_is_tall`), `miner404/pipeline.py` (router call,
both tilt gates, HTTP 504 `render_timeout` counted as a render timeout so a too-heavy module is sent back to be made
lighter, and the finishing steps), `miner404/versions.py` (plan). Everything else is the base verbatim.

## Local results vs the round-4 winner (VertexLoom)

The subnet's own multi-stage judge run locally (judge_service `evaluate_duel` @ 9fd95d9: S1 front match, then S2 / S3 when
S1 is a draw, S4 side guard), with assets rendered the Pod's way on a renderer built from the Pod's renderer-runtime stage,
DINOv3 ViT-S/16 embeddings and z-ai/glm-5.3-flash as the VLM. Margin = mean over 3 judge seeds of (wins - losses) / prompts.

| Set | H2e margin | Same judge, without the camera step |
|---|---|---|
| 16 dev prompts (round 4) | +37.5% | +16.7% |
| 16 random holdout prompts (round 4) | +58.3% | +58.3% |

All 32 final modules pass the official `miner-reference` validator. Budget per prompt in those runs: at most 56 calls and
4.36M credits (limits 60 / 5M); the camera step makes no model calls.
