# isExactEqual

[![Request Production Deploy](https://img.shields.io/badge/Request-Production_Deploy-2ea44f?style=for-the-badge)](https://github.com/lambda-feedback/IsExactEqual/issues/new?template=release-request.yml)

An evaluation function that checks exact equality between a student response and the correct answer. Both inputs are cast to a specified type before comparison using Python's `==` operator. The function is deployed on the [lambda-feedback](https://github.com/lambda-feedback) platform.

## Table of Contents
- [isExactEqual](#isexactequal)
  - [Table of Contents](#table-of-contents)
  - [Repository Structure](#repository-structure)
  - [Usage](#usage)
    - [Parameters](#parameters)
    - [Examples](#examples)
  - [How it works](#how-it-works)
    - [Docker & Amazon Web Services (AWS)](#docker--amazon-web-services-aws)
    - [Middleware Functions](#middleware-functions)
    - [GitHub Actions](#github-actions)
  - [Documentation](#documentation)
  - [Pre-requisites](#pre-requisites)
  - [Contact](#contact)

## Repository Structure

```
app/
    evaluation.py           # Main evaluation_function
    evaluation_tests.py     # Unit tests
    schema.json             # JSON schema for input validation
    requirements.txt        # Python dependencies
    Dockerfile              # Container image for AWS Lambda
    docs/
        user.md             # End-user documentation
        dev.md              # Developer reference

.github/
    workflows/
        test-lint.yml           # Run tests and linting on pull requests
        staging-deploy.yml      # Deploy to staging on push to main
        production-deploy.yml   # Deploy to production

config.json     # Evaluation function name
.gitignore
```

## Usage

Send a POST request to the deployed function endpoint with the following JSON body.

### Parameters

| Field | Type | Required | Description |
|---|---|---|---|
| `response` | any | Yes | The student's submitted response |
| `answer` | any | Yes | The correct answer |
| `params.type` | string | Yes | Cast type for both inputs before comparison. One of: `"int"`, `"float"`, `"str"`, `"dict"` |
| `params.display_submission_count` | boolean | No | If `true`, includes a submission count message in the feedback |
| `params.submission_context.submissions_per_student_per_response_area` | integer | No | Number of previously processed submissions for this student and response area |

### Examples

**Simple string comparison:**
```json
{
  "response": "hydrophobic",
  "answer": "hydrophobic",
  "params": {
    "type": "str"
  }
}
```
```json
{
  "is_correct": true
}
```

**Integer comparison:**
```json
{
  "response": "45",
  "answer": "45",
  "params": {
    "type": "int"
  }
}
```
```json
{
  "is_correct": true
}
```

**With submission count feedback:**
```json
{
  "response": "1",
  "answer": "1",
  "params": {
    "type": "int",
    "display_submission_count": true,
    "submission_context": {
      "submissions_per_student_per_response_area": 2
    }
  }
}
```
```json
{
  "is_correct": true,
  "feedback": "You have submitted 3 responses."
}
```

---

## How it works

The function is built on top of a custom base layer, [BaseEvaluationFunctionLayer](https://github.com/lambda-feedback/BaseEvalutionFunctionLayer), which provides tooling, testing helpers, and schema checking common to all evaluation functions.

### Docker & Amazon Web Services (AWS)

The evaluation function is hosted on AWS Lambda and packaged as a Docker container. Docker bundles the function and all its dependencies into a single image, which AWS runs on demand inside a container. For background reading, see this [introduction to containerisation](https://www.freecodecamp.org/news/a-beginner-friendly-introduction-to-containers-vms-and-docker-79a9e3e119b/) and the [AWS Lambda documentation](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html).

### Middleware Functions

Middleware provided by [BaseEvaluationFunctionLayer](https://github.com/lambda-feedback/BaseEvalutionFunctionLayer) handles request validation, error formatting, and response serialisation. The `evaluation.py` file only needs to implement the core comparison logic.

### GitHub Actions

Three pipelines are configured in `.github/workflows/`:

- **`test-lint.yml`** — runs on every pull request: executes the unit test suite and linting via `flake8`.
- **`staging-deploy.yml`** — runs on every push to `main`: re-runs tests, then builds the Docker image, pushes it to the shared ECR repository, and deploys to the staging environment.
- **`production-deploy.yml`** — promotes a tested build to the production environment.

## Documentation

- [`app/docs/user.md`](app/docs/user.md) — end-user guide explaining inputs and parameters
- [`app/docs/dev.md`](app/docs/dev.md) — developer reference with detailed input/output specs and examples

## Pre-requisites

To develop and test locally:

- Python 3.8 or higher
- Docker
- `git` CLI or GitHub Desktop
- A code editor (VS Code, PyCharm, etc.)

## Contact

For questions or issues, open a GitHub issue or visit the [lambda-feedback organisation](https://github.com/lambda-feedback).