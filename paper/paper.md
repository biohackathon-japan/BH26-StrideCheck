---
title: 'DBCLS BioHackathon 2026 report: StrideCheck, a browser-only prototype that turns running videos into gait measures, joint-load estimates and an honest injury prediction'
title_short: 'BioHackJP26: StrideCheck'
tags:
  - Running biomechanics
  - Running-related injury
  - Pose estimation
  - Open data
  - Web application
authors:
  - name: Keitaro Takaoki
    affiliation: 1
  - name: Haruma Abe
    affiliation: 1
  - name: Teppei Okazaki
    affiliation: 1
affiliations:
  - name: Tokyo City University, Tokyo, Japan
    ror: 04dt6bw53
    index: 1
date: 18 September 2026
cito-bibliography: paper.bib
event: BH26JP
biohackathon_name: "DBCLS BioHackathon 2026"
biohackathon_url:   "https://2026.biohackathon.org/"
biohackathon_location: "Matsuyama, Japan, 2026"
group: StrideCheck
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/biohackathon-japan/BH26-StrideCheck
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: Keitaro Takaoki \emph{et al.}
---


# Introduction

Running-related injuries are common, and gait analysis that could flag
risky running form usually needs a laboratory.  Smartphone video is
available to every runner, but two questions decide whether it is useful:
how well can running form be measured from ordinary video, and how much do
those measures actually tell about injury or joint load?

During the DBCLS BioHackathon 2026 we built StrideCheck, a prototype web
page that answers both questions openly.  From a side-view and a front- or
rear-view video it (1) describes the runner's rhythm and form as types
placed on the distribution of published cohorts, (2) gives a form-based
injury prediction with its uncertainty and its (low) accuracy, and (3)
estimates the load on four body regions relative to an average runner.  The
video is analysed entirely in the browser and never uploaded.  Every model
was built from open data, and the video measurement was validated against
motion capture and force plates recorded with the same videos.

# Data

We used four public datasets to build the models and two to validate the
video measurement (Table 1).

| Dataset | Content | Used for |
|---|---|---|
| Wu et al. 2026 [@citesAsDataSource:Wu2026] | 142 endurance runners, 12-month prospective follow-up, weekly injury records, force-based gait timing at baseline | Rhythm types; rhythm-based prediction |
| Loh et al. 2025 [@citesAsDataSource:Loh2025] | 81 runners (26 injured within 12 months), 2-D video angles at baseline | Form types; form-based prediction |
| Loh & Kong 2026 [@citesAsDataSource:LohKong2026] | 154 injured and 44 uninjured runners, 2-D video angles | Angle definitions and distributions |
| Fukuchi et al. 2017 [@citesAsDataSource:Fukuchi2017; @citesAsDataSource:Fukuchi2017data] | 39 runners, treadmill 2.5-4.5 m/s, markers and force plate | Joint-load models; validation of event detection |
| Wang et al. 2022-2023 [@citesAsDataSource:Wang2022data; @citesAsDataSource:Wang2022video; @citesAsRelated:Wang2023] | Treadmill running at three speeds (6.3-9.9 km/h), front and side video (33 fps) recorded with Qualisys markers and a split-belt force treadmill | Validation of the video measurement (24 trials, 9 runners) |

Table: Datasets.  The Loh datasets are CC BY-NC; the others are CC BY 4.0
or were used for validation only and are not redistributed.

# Methods

## Types and injury prediction

To test whether types carry injury information, runners were clustered on
cadence, duty factor, left-right asymmetry and foot strike (Wu et al.) and
on form angles (Loh et al.).  For display, the page uses simpler types:
cadence by foot strike, and contralateral pelvic drop by knee flexion at
mid-stance, each split at the weighted median of the cohort.  The types
only describe a runner; they are not used for prediction.  Injury
prediction uses the measured values directly (logistic models on pelvic
drop, hip adduction, knee flexion and knee valgus), evaluated with
person-wise cross-validation and bootstrap confidence intervals.

## Joint load

From the Fukuchi markers we computed the values a video would give
(markers projected to 2-D) and fitted ridge models for peak joint moments
and ground reaction forces.  For each region we report how much the video
values explain beyond running speed and body size ($R^2_{extra}$) as a
reliability rating.

## Video measurement in the browser

The page runs MediaPipe Pose Landmarker Heavy [@usesMethodIn:Bazarevsky2020]
in the browser.  The frame rate is read from the MP4 header, and every frame
is analysed once.  Initial contact is the heel's most forward point
relative to the pelvis plus 29 ms, and toe-off is the toe's most rearward
point minus 12% of the stride [@usesMethodIn:Zeni2008]; both offsets were
fitted against force plates.  In the side view the pose model often labels
the legs by position (front and back) rather than by side, so the labels
swap at every step and the far leg's points are pulled onto the near leg
when the legs cross.  StrideCheck tracks the two legs through each crossing
(one crossing per step), predicts their path through the crossing frames,
and names and corrects them step by step against the front or rear video,
where left and right are reliable.  A check view draws the estimated joints
over the video so that users can see these steps.

## Validation

Event detection was first checked on the Fukuchi markers, laid out as pose
landmarks (hip joint centre after Harrington et al.
[@usesMethodIn:Harrington2007]), at 33-120 fps with added noise.  The whole
pipeline was then checked on the Wang et al. videos: the Qualisys recording
starts with the videos (offset 3 ms, median), the force treadmill gives the
true contacts of each leg, and the markers give the true angles, computed
with the same 2-D formulas.

# Results

**Types and prediction.** Neither rhythm types nor form types differed in
injury rate (three rhythm clusters: 67%, 88% and 85% injured within 52
weeks, log-rank p = 0.20; form types: 28% vs 36%, p = 0.44).  The form-based prediction reached an AUC of
0.64 [0.52-0.76]; the rhythm-based one, 0.58 [0.39-0.75], cannot be told
apart from chance.  Adding injury history or training volume made
prediction worse.  The published AUC of 0.76 for the Wu et al. data was
reproduced only when rows were split at random; split by person, it fell to
0.48.

**Joint load.** Beyond speed and body size, the video values explained part
of the load for the shin ($R^2_{extra}$ 0.37), the patellofemoral region
(0.30) and the lateral hip (0.15), but not the Achilles tendon (-0.07).  A
10% higher cadence corresponded to 8.7% lower knee load and 6.4% lower hip
load.

**Video measurement.** Table 2 compares the browser pipeline with the
ground truth.  Leg tracking raised the share of steps with the correct
left/right label from 91% (the pose model's own labels) to 98%.  Initial contact was
unbiased on both legs.  Contact time, overstride and the near-leg foot
angle at contact followed the truth closely.  Knee flexion at mid-stance
did not: runners differ in it by only 6.8° (SD), less than the measurement
error, so the page does not report it and uses the cohort median instead.

| Measure (per trial and leg) | Near leg | Far leg |
|---|---|---|
| Initial contact per step | -1 ± 16 ms | +2 ± 12 ms |
| Contact time | bias -6 ms, SD 29 ms, r = 0.83 | bias -4 ms, SD 20 ms, r = 0.91 |
| Foot angle at contact | +5.3°, SD 4.1°, r = 0.85 | +16.1°, SD 9.1°, r = 0.61 |
| Overstride | r = 0.93 | r = 0.94 |
| Knee flexion at mid-stance | -16.6°, r = 0.48 | -14.7°, r = -0.01 |

Table: Browser pipeline (MediaPipe Heavy) against motion capture and force
plates, 24 trials; the side camera filmed the left side, so the right leg is
the far leg.

# Discussion

StrideCheck shows how far ordinary video and open data go today.  Timing
measures and overstride can be measured from 33 fps video with errors that
are small relative to differences between runners, whereas knee angles
cannot.  The injury predictions built on these measures are weak, and the
page states their accuracy next to every number.

Limitations: the contact-time error (20-29 ms) is larger than the 10 ms
target, which needs 120 fps video; the front video was used in place of a
rear view; the far-leg foot angle is biased; one setting of the leg
correction was chosen on the validation trials; and the injury cohorts are
small.  StrideCheck is a research prototype, not medical advice.

The next step is to turn the page into an entry point for prospective open
data: runners who consent would share their measured values and later
injuries, so that the models can be rebuilt on data that match the way the
page measures.

## Availability

The page runs at <https://keitaro0510.github.io/stride-check/>.  Its source is
at <https://github.com/Keitaro0510/stride-check>, and the analysis and
validation code at <https://github.com/Keitaro0510/stride-check-analysis>, both
under the MIT License.  Third-party data are not redistributed; values
derived from them keep the source licenses (non-commercial for the Loh et al.
data).

## Acknowledgements

We thank the organizers of the DBCLS BioHackathon 2026, and the authors of
the datasets we used for making their data openly available.

# References
