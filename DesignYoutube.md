# Design youtube interview question

## Table of content

- [Design youtube interview question](#design-youtube-interview-question)
  - [Table of content](#table-of-content)
  - [Follow up Questions](#follow-up-questions)
  - [Requirements](#requirements)
  - [API Design](#api-design)

## Follow up Questions

Questions to Ask the **Interviewer** before you start doing anything.

**Candidate**: What scale are we designing for?

**Interviewer**: 1 million uploads per day, 100 million DAU²

**Why it matters**: Scale drives sharding, partitioning, and caching

**Interview insight**: You show you understand capacity planning

===

**Candidate**: What’s the read-to-write ratio?

**Interviewer**: About 100 views for every 1 upload

**Why this matters**: It’s a read-heavy system, so the design favors streaming

**Interview insight**: You show awareness of workload patterns

===

**Candidate**: What’s the maximum video file size?

**Interviewer**: 256 GB

**Why this matters**: Forces you to use multipart uploads and chunking

**Interview insight**: You consider edge cases

===

**Candidate**: What video formats and resolutions do we support?

**Interviewer**: MP4, AVI, MOV, and from 240p to 4K

**Why this matters**: Needs a strong transcoding pipeline with many output renditions

**Interview insight**: Shows attention to real-world constraints

===

**Candidate**: What’s the target latency for streaming?

**Interviewer**: First frame in under 500ms

**Why this matters**: Requires CDN⁷ and adaptive streaming

**Interview insight**: You focus on user experience

===

**Candidate**: What’s acceptable for upload processing time?

**Interviewer**: Around 10 to 30 minutes

**Why this matters**: Processing can be asynchronous

**Interview insight**: You understand eventual consistency

===

**Candidate**: Is eventual consistency fine for new uploads?

**Interviewer**: Yes, a short delay is acceptable

**Why this matters**: Availability over strict immediacy

**Interview insight**: You show CAP theorem reasoning

===

**Candidate**: What’s our uptime target?

**Interviewer**: 99.9%

**Why this matters**: Need redundancy and failover

**Interview insight**: You think about reliability

===

**Candidate**: Do we support livestreaming?

**Interviewer**: For this interview, focus on uploads

**Why this matters**: Keeps scope controlled

**Interview insight**: You avoid feature creep

## Requirements

Be also careful for non technical requirements: fast access, scalability, new uploads can have eventual consistency.

## API Design

You create an API with endpoints and post and get videos. API for each user to upload view progress. Endpoint to search the videos whose name / author matches a given user input.

Transcode videos in a common format that we store. We also keep the original videos. We don't transcode unused videos in HD formats in order to avoid taking space. We regularly check video popularity / views in order to free unpopular videos from CDN.
