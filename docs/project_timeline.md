# HOMR-Based Music Recognition Application --- Project Timeline

**Planning date:** 19 November 2026\
**MVP target:** 31 December 2026\
**Project context:** Bachelor's thesis application using HOMR for
optical music recognition, with an Angular frontend, Spring Boot
backend, PostgreSQL database, and a Python service for HOMR and music
analysis.

## Project goal

Deliver a working, demonstrable application by **31 December 2026**. The
December milestone is an integrated MVP: users can upload sheet-music
images or PDFs, process them with HOMR, view the recognized score,
inspect basic musical analysis, and download the generated MusicXML.

After December, keep the application's feature scope largely frozen. Use
the remaining thesis period for measured AI experiments, evaluation, bug
fixes, documentation, and presentation polish rather than adding major
product features.

## Target architecture

-   **Frontend:** Angular + TypeScript; Angular Material for UI
    components.
-   **Backend:** Spring Boot; REST API, business logic, job management,
    and persistence.
-   **Database:** PostgreSQL for score metadata, processing jobs, and
    analysis results.
-   **AI service:** Python + FastAPI, wrapping HOMR and supporting
    preprocessing/analysis libraries.
-   **Recognition:** HOMR remains the OMR engine; do not replace it.
-   **Music analysis:** music21 for extracting musical information from
    MusicXML.
-   **Score rendering:** OpenSheetMusicDisplay (OSMD) in Angular.
-   **File storage:** local filesystem for the MVP; store file paths and
    metadata in PostgreSQL.
-   **Service communication:** Spring Boot calls the Python service over
    an internal REST API.
-   **Development environment:** use WSL2 for HOMR development/testing
    on Windows if native Windows compatibility becomes a blocker.

Keep HOMR as a separate checkout or Git submodule rather than copying
its source into the application. Before distributing or publicly hosting
the application, review HOMR's AGPL-3.0 licensing obligations.

## December MVP scope

### Must-have features

-   Upload JPG, PNG, and PDF sheet-music files.
-   Validate file types and sizes and handle invalid uploads clearly.
-   Store original uploads and relevant metadata.
-   Submit recognition jobs to the Python/HOMR service.
-   Track job states: `PENDING`, `PROCESSING`, `COMPLETED`, and
    `FAILED`.
-   Save generated MusicXML and make it available for download.
-   Display the recognized score in the browser using OSMD.
-   Show basic analysis where the recognized MusicXML supports it:
    -   key signature;
    -   time signature;
    -   notes and durations;
    -   measure count;
    -   pitch range and melody contour;
    -   basic rhythm distribution;
    -   basic chord identification, where reliable.
-   Provide a score history/list and a score details page.
-   Include useful error messages, logging, and setup documentation.

### Explicitly out of scope for the December MVP

-   Training a new OMR model from scratch.
-   Replacing HOMR's recognition architecture.
-   Large-scale redesign of HOMR's transformer.
-   Advanced music-theory analysis that cannot be validated reliably.
-   Major product features added after the MVP deadline.

## Weekly implementation plan

### 19--22 November --- Setup and feasibility

-   Create the repository structure and document the intended
    architecture.
-   Create minimal Angular and Spring Boot projects.
-   Set up the Python environment and run HOMR against a few
    representative sample images.
-   Confirm that HOMR produces usable MusicXML.
-   Decide how the Python service will expose recognition to Spring
    Boot.
-   Record any environment issues and choose a repeatable development
    setup.

**Deliverable:** all three application parts start locally, and HOMR has
been run successfully on sample input.

### 23--29 November --- Backend foundation

-   Configure PostgreSQL and Spring Data JPA.
-   Create initial entities:
    -   `Score`
    -   `ProcessingJob`
    -   `AnalysisResult`
    -   `GeneratedFile` (if a separate file record is useful)
-   Implement upload validation and original-file storage.
-   Implement basic score and job REST endpoints.
-   Create a minimal FastAPI wrapper around HOMR.
-   Define the request/response contract between Spring Boot and Python.

**Deliverable:** a file can be uploaded, recorded in the database, and
submitted to the Python service in a basic end-to-end test.

### 30 November--6 December --- HOMR integration and processing pipeline

-   Connect Spring Boot to the Python service over REST.
-   Process JPG/PNG/PDF inputs through HOMR.
-   Save generated MusicXML and processing metadata.
-   Validate that MusicXML output is present and parseable.
-   Add initial OpenCV preprocessing only where it helps input
    reliability.
-   Use music21 to extract a first set of musical properties.
-   Implement asynchronous processing so a long recognition task does
    not keep an HTTP request open.
-   Persist job state transitions and errors.

**Deliverable:** an uploaded score can be processed asynchronously and
produce a stored MusicXML result.

### 7--13 December --- Frontend and score display

-   Implement upload UI, including drag-and-drop if time allows.
-   Show upload/processing progress and job status.
-   Implement score history and score details views.
-   Render generated MusicXML with OSMD.
-   Add MusicXML download.
-   Handle empty, loading, failed, and completed states.

**Deliverable:** a user can upload a score, follow its processing
status, view the recognized notation, and download the MusicXML.

### 14--20 December --- Musical analysis and visualizations

-   Extract key signature and time signature.
-   Extract notes, durations, measure count, and pitch range.
-   Add melody contour and rhythm summaries.
-   Attempt basic chord identification where the input supports it.
-   Add a small number of clear charts or visual summaries.
-   Distinguish the notated key signature from any estimated tonal
    center; they are not necessarily the same.

**Deliverable:** the score page includes useful, clearly labelled basic
musical analysis.

### 21--27 December --- Integration, testing, and reliability

-   Test the complete workflow with several representative scores.
-   Test malformed files and malformed or incomplete MusicXML.
-   Add upload size/type limits and useful error messages.
-   Improve logging and diagnose failed processing jobs.
-   Add unit and integration tests for critical backend and
    Python-service behavior.
-   Add an end-to-end smoke test for upload → recognition → display →
    download.
-   Fix integration and UI issues; avoid expanding the feature list.

**Deliverable:** a stable MVP that works on a small, documented set of
test scores.

### 28--31 December --- Freeze and release the MVP

-   Fix remaining high-priority bugs.
-   Verify the complete demo workflow from a clean start.
-   Include a few sample scores for demonstration.
-   Write setup instructions and a short architecture overview.
-   Document known limitations and failure cases.
-   Tag the repository release, for example `v1.0-mvp`.

**Deliverable:** a reproducible, documented, demonstrable MVP by 31
December 2026.

## Suggested API outline

  -----------------------------------------------------------------------------
  Method                  Endpoint                      Purpose
  ----------------------- ----------------------------- -----------------------
  `POST`                  `/api/scores`                 Upload a score and
                                                        create its record

  `GET`                   `/api/scores`                 List uploaded scores

  `GET`                   `/api/scores/{id}`            Get score details

  `DELETE`                `/api/scores/{id}`            Delete a score and its
                                                        associated files

  `POST`                  `/api/scores/{id}/process`    Start or retry
                                                        recognition

  `GET`                   `/api/jobs/{id}`              Get processing status

  `GET`                   `/api/scores/{id}/analysis`   Get extracted musical
                                                        analysis

  `GET`                   `/api/scores/{id}/musicxml`   Download generated
                                                        MusicXML
  -----------------------------------------------------------------------------

Treat this as an initial contract; adjust endpoint names and resource
relationships as the implementation develops.

## Suggested database responsibilities

-   **Score:** original filename, stored path, upload timestamp, MIME
    type, and overall status.
-   **ProcessingJob:** score reference, job state, start/end timestamps,
    error details, and processing version.
-   **AnalysisResult:** extracted properties and analysis version.
-   **GeneratedFile:** generated file type, stored path, creation
    timestamp, and associated score/job.

Keep the actual image/PDF and MusicXML files on disk for the MVP;
PostgreSQL should store their metadata and paths rather than large file
contents.

## January--May 2027 --- Evaluation, AI experiments, and thesis polish

The application feature set should remain mostly frozen after the
December MVP. Later work should focus on improving and evaluating
recognition quality, then presenting the results clearly.

### January --- Establish a baseline

-   Build a fixed evaluation set of representative score images.
-   Run the unmodified HOMR pipeline on the set.
-   Record recognition quality, processing time, and failure cases.
-   Categorize errors such as pitch, rhythm, accidentals, rests, beams,
    and staff detection.
-   Save baseline outputs so later experiments can be compared fairly.

**Deliverable:** a documented baseline and a prioritized list of
recognition weaknesses.

### February --- Preprocessing experiments

-   Experiment with deskewing and perspective correction.
-   Test denoising, contrast enhancement, and adaptive thresholding.
-   Investigate staff-line normalization/removal only if justified by
    observed errors.
-   Evaluate each change on the same fixed dataset.
-   Keep changes that measurably help; record regressions as well.

**Deliverable:** a small, measured set of preprocessing improvements or
evidence that they do not help.

### March --- One focused recognition improvement

-   Select one significant error category from the baseline.
-   Investigate whether it can be improved through existing HOMR
    components, targeted model fine-tuning, or a focused preprocessing
    change.
-   Avoid attempting a wholesale transformer redesign.
-   Keep experiments reproducible and record configuration/model
    versions.

**Deliverable:** one clearly scoped AI improvement experiment.

### April --- Compare and integrate

-   Compare baseline and improved versions on the same evaluation set.
-   Report improvements, regressions, and remaining limitations.
-   Integrate the best-performing version behind a clear
    configuration/version choice.
-   If practical, make it possible to compare original and improved
    results.

**Deliverable:** final comparison results and an integrated best
version.

### May --- Final polish and thesis preparation

-   Fix high-priority usability and reliability issues.
-   Polish the score and analysis presentation without adding major
    features.
-   Finalize setup, architecture, and experiment documentation.
-   Prepare methodology, evaluation, results, limitations, and demo
    material for the thesis.
-   Rehearse the complete demonstration.

**Deliverable:** a polished application, reproducible experiments, and
thesis-ready documentation/results.

## Risk management and working rules

1.  **Keep the MVP small.** A complete upload-to-MusicXML workflow is
    more valuable than many unfinished features.
2.  **Integrate HOMR early.** Do not postpone testing the real
    recognition pipeline until the frontend is finished.
3.  **Separate responsibilities.** Spring Boot manages application
    workflows; Python handles OMR and Python-based analysis.
4.  **Use asynchronous jobs.** Recognition may take too long for a
    normal synchronous request.
5.  **Preserve a baseline.** Do not change the recognition pipeline
    before recording how the original performs.
6.  **Measure AI changes.** Compare against the same fixed evaluation
    set instead of relying on a few visually pleasing examples.
7.  **Avoid risky core changes initially.** Start with integration and
    observable preprocessing opportunities before changing HOMR's core
    model.
8.  **Freeze scope at the end of December.** After the MVP, prioritize
    evaluation, focused AI experiments, bug fixes, and thesis
    presentation.

## Definition of done for 31 December 2026

The MVP is considered complete when:

-   [ ] The application can be started using documented instructions.
-   [ ] A user can upload a supported image or PDF.
-   [ ] The backend stores the upload and creates a processing job.
-   [ ] The Python service runs HOMR and returns a MusicXML result.
-   [ ] Job status and processing errors are persisted and visible.
-   [ ] The frontend renders the resulting score.
-   [ ] The user can download the MusicXML.
-   [ ] Basic musical analysis is displayed for supported outputs.
-   [ ] The main workflow has been tested with multiple sample scores.
-   [ ] Known limitations and setup steps are documented.
-   [ ] A repeatable demonstration can be run from a clean start.
