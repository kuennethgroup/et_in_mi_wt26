# Emerging Trends in Materials Informatics (ETinMI) 

**Course Exercises - Winter Term 2026 / 2027**  
**Instructor**: Christopher Kuenneth  
**Computational Materials Science, University of Bayreuth**  


## Overview

Welcome to the exercises for the "Emerging Trends in Materials Informatics" course! Below you'll find relevant instructions, tools, and notes to get started effectively with the coursework.

### Python Dependency Management

This repo utilizes [uv](https://docs.astral.sh/uv/getting-started/installation/) for managing and tracking Python dependencies. Make sure to set up uv to easily manage environments and dependencies for working on the exercises. If you work on Coder, uv is already installed.

### Materials Project API key

The Materials Project notebook needs **your own** free API key:

1. Log in at [materialsproject.org](https://materialsproject.org) with your university e-mail address.
2. Copy your API key from your [dashboard](https://materialsproject.org/dashboard).
3. Run the notebook. It asks for the key in a hidden prompt. Alternatively, set the environment variable
   `MP_API_KEY` before starting VSCode/Jupyter.

**Never paste your key into a notebook cell or commit it to Git.**
If you have no key yet (or the API is down), set `USE_CACHED = True` in the first code cell of Part B; the notebook then uses
the cached data in `materials_apis/data/`. NOMAD needs no key.

Data from the Materials Project and NOMAD are licensed under CC BY 4.0; cite the databases when you use them.



