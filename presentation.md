## Gap actions

Sergio Siccha

<small>
TU Kaiserslautern
</small>

<small>Created with [reveal.js](https://revealjs.com)</small>

---

### Documentation for GitHub Actions

[https://docs.github.com/en/actions](https://docs.github.com/en/actions)

---

<img data-src="gap-actions-orga.png"
     alt="screenshot of the gap-actions github organization page"
     width="100%">

---

### Documentation artifacts

[gap-packages/recog](https://github.com/gap-packages/recog/actions/runs/428260043)

---

### Demo package
[ssiccha/TestActionPackage](https://github.com/ssiccha/TestActionPackage)

---

### Yaml

- map/record
  ```yaml
  bread: 5
  butter: 1000
  ```
- list
  ```yaml
  - 5
  - 1000
  ```

---

### A minimal CI file
```yaml[1-5|7-16]
name: CI

on:
  - push
  - pull_request

jobs:
  # The CI test job
  test:
    name: CI test
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - uses: gap-actions/setup-gap-for-packages@v1
      - uses: gap-actions/run-test-for-packages@v1
```

---

### Code coverage for free

<small>
... if your repository is public.
</small>

[codecov](https://app.codecov.io/gh/ssiccha/TestActionPackage)

---

### Only PR and push main
```yaml[12-16]
name: CI

# Trigger the workflow on push or pull request
on:
  push:
    branches:
      - main # change this to 'master' if necessary!
  pull_request:

jobs:
  # The CI test job
  test:
    name: CI test
    runs-on: ubuntu-latest
    # Don't run this twice on PRs for branches pushed to the same repository
    if: ${{ !(github.event_name == 'pull_request' && github.event.pull_request.head.repo.full_name == github.repository) }}

    steps:
      - uses: actions/checkout@v2
      - uses: gap-actions/setup-gap-for-packages@v1
      - uses: gap-actions/run-test-for-packages@v1
```

---

### A documentation job

```yaml[19-34]
name: CI

# Trigger the workflow on push or pull request
on:
  - push
  - pull_request

jobs:
  # The CI test job
  test:
    name: CI test
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - uses: gap-actions/setup-gap-for-packages@v1
      - uses: gap-actions/run-test-for-packages@v1

  # The documentation job
  manual:
    name: Build manuals
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - uses: gap-actions/setup-gap-for-packages@v1
      - uses: gap-actions/compile-documentation-for-packages@v1
        with:
          use-latex: 'true'
      - name: 'Upload documentation'
        uses: actions/upload-artifact@v1
        with:
          name: manual
          path: ./doc/manual.pdf
```

---

### Some options

```yaml[11-21|23-30]
name: CI

# Trigger the workflow on push or pull request
on:
  - push
  - pull_request

jobs:
  # The CI test job
  test:
    name: ${{ matrix.gap-branch }} - HPCGAP ${{ matrix.HPCGAP }}
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        gap-branch:
          - master
          - stable-4.11
        HPCGAP:
          - yes
          - no

    steps:
      - uses: actions/checkout@v2
      - uses: gap-actions/setup-gap-for-packages@v1
        with:
          GAPBRANCH: ${{ matrix.gap-branch }}
          HPCGAP: ${{ matrix.HPCGAP }}
          GAP_PKGS_TO_BUILD: 'io orb profiling'
      - uses: gap-actions/run-test-for-packages@v1

  # The documentation job
  manual:
    name: Build manuals
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - uses: gap-actions/setup-gap-for-packages@v1
      - uses: gap-actions/compile-documentation-for-packages@v1
        with:
          use-latex: 'true'
      - name: 'Upload documentation'
        uses: actions/upload-artifact@v1
        with:
          name: manual
          path: ./doc/manual.pdf
```

---

### matrix.include

```yaml[19-22]
name: CI

# Trigger the workflow on push or pull request
on:
  - push
  - pull_request

jobs:
  # The CI test job
  test:
    name: ${{ matrix.gap-branch }} - HPCGAP ${{ matrix.HPCGAP }}
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        gap-branch:
          - master
          - stable-4.11
        HPCGAP: ['no']
        include:
          - gap-branch: master
            HPCGAP: 'yes'

    steps:
      - uses: actions/checkout@v2
      - uses: gap-actions/setup-gap-for-packages@v1
        with:
          GAPBRANCH: ${{ matrix.gap-branch }}
          HPCGAP: ${{ matrix.HPCGAP }}
          GAP_PKGS_TO_BUILD: 'io orb profiling'
      - uses: gap-actions/run-test-for-packages@v1

  # The documentation job
  manual:
    name: Build manuals
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - uses: gap-actions/setup-gap-for-packages@v1
      - uses: gap-actions/compile-documentation-for-packages@v1
        with:
          use-latex: 'true'
      - name: 'Upload documentation'
        uses: actions/upload-artifact@v1
        with:
          name: manual
          path: ./doc/manual.pdf
```


---

### More options

See the `README.md` files of:
- [gap-actions/setup-gap-for-packages](https://github.com/gap-actions/setup-gap-for-packages)
- [gap-actions/run-test-for-packages](https://github.com/gap-actions/run-test-for-packages)
- [gap-actions/compile-documentation-for-packages](https://github.com/gap-actions/compile-documentation-for-packages)

---


### Badges
[README.md](https://github.com/ssiccha/TestActionPackage)
```markdown
[![CI](https://github.com/ssiccha/TestActionPackage/workflows/CI/badge.svg)](https://github.com/ssiccha/TestActionPackage/actions?query=workflow%3ACI+branch%3Amain)
[![Code Coverage](https://app.codecov.io/github/ssiccha/TestActionPackage/coverage.svg?branch=main&token=)](https://app.codecov.io/gh/ssiccha/TestActionPackage)
```

<small>
May have to do `branch main` -> `branch master`.
</small>

---

### SSH onto the runner

[Debugging with tmate](https://github.com/marketplace/actions/debugging-with-tmate)

```yaml
# A debugging step
- name: "Start SSH if a previous step failed"
  if: ${{ failure() }}
  uses: mxschmitt/action-tmate@v3
  timeout-minutes: 15
```

---

<small>
Background image:

Florian Maderebner,
[Two Man Hiking on Snow Mountain](https://www.pexels.com/photo/two-man-hiking-on-snow-mountain-869258/)
(Pexel license)
</small>
