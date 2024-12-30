# **Building Spokane Tech: Part 2***

Welcome to part 2 of the "Building Spokane Tech" series! In this article, we walk though the layout of the repository and detail what code lives where.


## **Repository Structure**

Below is an overview of the repository structure.

```plaintext
.
├── src
│   ├── django_project
│   │   ├── core
│   │   │   └── ...
│   │   ├── tests
│   │   │   ├── integration
│   │   │   │   └── ...
│   │   │   ├── regression
│   │   │   │   └── ...
│   │   │   └── unit
│   │   │       └── ...
│   │   ├── web
│   │   │   └── ...
│   │   └── manage.py
│   ├── docker
│   │   └── Dockerfile
│   └── envs
│       └── .env.template
├── .gitignore
├── LICENSE
├── README.md
└── pyproject.toml
```

## **File and Directory Descriptions**

### The Root Directory
Our root directory is lean and clean, with only the standard files for a git repo, including a .gitignore, a LICENSE file, a README.md file, and a pyproject.toml. 
The pyproject.toml file contains our dependencies and tool configurations. Source code is located under the src directory.

### The src Directory
All of our actual code lives under the src directory. The are subdirectories for the django code, docker files, and our environment files.

#### django_project
The django_project directory has ***core***, the django project level directory, ***web***, the django app directory, and ***tests***, the location of all django unittests, with further subdirectories for test categories.

#### docker
The docker directory will have applicable docker related files such as the Dockerfile, docker-compose.yaml, and .dockerignore. 

#### envs
Any applicable .env files live here. There is a .env.template, .env.local, and other .env files as needed. Note, most .env files will not be checked into source control.
