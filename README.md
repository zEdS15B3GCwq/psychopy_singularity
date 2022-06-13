# psychopy_singularity
PsychoPy installation in Singularity container

### Work in progress

This project aims to create a fully functional PsychoPy (PP) installation in a Singularity container.

Basic functions seem to be working, but not much testing has been done.

This is a personal project, only tested on my computer, so tweaking may be necessary to get it in a working state elsewhere. No guarantees of quality given. Documentation is minimal.

### Technical details in brief

Developed on/for Ubuntu 22.04 (both host & container).

For now, a Docker container is set up to run PP, then converted into a Singularity container. Docker is more tolerant to frequent changes and rebuilds, while Singularity seems to redo all the build steps from zero, so this combination appears to be better for development.

**Key points:**
* `Ubuntu:22.04` base
* Build & install Python 3.8.13 (3.8 recommended for PP?)
* Install PP into *virtualenv*. Extra packages required by `pocketsphinx` and `wxpython` (latter might have wheels available later)
* Copy *virtualenv* (to `/venv`) and Python (to `/python`) to clean Ubuntu 22.04 image (2nd stage)
* Install PP requirements (`apt` packages)
* Fix ownership of *virtualenv* to allow non-root execution (for now, assumes user:group IDs 1000:100; won't work if launched with other user)

### How to use

1. Install Docker and Singularity
2. Do you need NVidia container toolkit and/or other stuff needed for containers?
3. setup PP security limits for PsychToolBox group (google PP installation instructions for Ubuntu or just launch PP and read the hint)
4. `sudo docker build -t pp:latest .` in folder with `Dockerfile`
5. `sudo singularity build pp.sif pp.def` in folder with `pp.def`
6. `singularity run --bind /run --bind /etc/security/limits.d` ; `--nv` if needed? (see **Known issues** on user identity)
7. `. /venv/bin/activate`
8. launch PP (e.g. `psychopy -c`)

### Feedback is welcome

I need input to be able to make PP fully functional; my knowledge on what PP really requires, and I can't test if everything works well, alone.

### Known issues, questions

* How to set up permissions (e.g. for *virtualenv*) if container user is undefined? Singularity container user has the same IDs as the one who runs the container, which is unpredictable. For now, it is assumed that the user has IDs 1000:1000. -> solutions: fakeroot? sudo singularity run --security uid:x --security gid:y?
* PP complains about not being able to set the locale to US -> this is probably not a big issue just haven't had time to look into it yet
* Haven't been able to test sound stimuli from within UI (didn't find appropriate demo script). Sound works from Python code (`from psychopy import sound; s=sound.Sound(); s.play()`)
- performance? PTB really working?
- How to test if psychtoolbox group + sec limits have an effect in conatainer?
- By default, Singularity containers have a read-only file system; PP throws an error here and there -> persistent overlay? writeable bind mount?
