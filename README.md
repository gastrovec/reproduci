# REPRODUCI

Version 0.1

*reproduci* ([re.pro'du.ʧ]) is a methodology for reproducible research.

## Motivation

Much of our research emerges experimentally, and over long periods of time
(often months). Without taking care to make it reproducible early on, often we
will find ourselves unable to remember how some file was created, having to
guess, look through our shell's history file, or just hope that noone will ask.
This is a sad state of affairs.

## Previous Work

Other systems aiming to make research reproducible often are specific to one
framework, or language. *reproduci* aims to be minimal and agnostic to such
specifics.

## Folder structure

*reproduci* enforces a certain folder structure:

| In VCS? | Name         | Description    | Permissions |
|---------|--------------|----------------|-------------|
| ?       | `sources/`     | Original files | `r--r--r--`   |
| ✖︎       | `outputs/`     | Final outputs  | `r--r---w-`   |
| ✖︎       | `workdata/`    | Intermediaries | `r--rw----`   |
| ✔︎       | `scripts/`     | Programs, etc. | `rw-r-----`   |
| ✔︎       | `persistent/`  | Annotations    | `rw-r-----`   |
| ✔︎       | `writing/`     | LaTeX sources  | `rw-r-----`   |  # FIXME
| ✔︎       | `Makefile`     | A Makefile     | `rwx------`   |

The permissions in this table can be read like unix permissions, with the user
being the human, group representing their programs (in `scripts/`), and world
meaning anything outside this project.


### `sources/`

This is where your input data goes, in an unmodified form. Typically, that is
corpora, image data, datasets, etc.

Ideally, this is checked into the repository, but that is sometimes impossible
because of size, or legal issues. The second-best option is providing some kind
of `setup` target in your Makefile that automatically downloads and sets up the
data in this folder (that would be the one exception to the restriction on
scripts writing to this folder).

Files in this directory should be treated as immutable, and any transformations
should be made either in memory during processing, or through a copy in
`workdata/`.


### `outputs/`

This is where the final outputs of the project go. In the best case, that is an
automatically generated paper, but things like plots also belong here (unless
they're used inside a paper). Numerical results should also go here, but there
is an additional mechanism for those, as described in the section FIXME.


### `workdata/`

Usually this will be the messiest folder: It is where every intermediary file
goes, i.e. anything that is produced from the `sources/` and needed to produce
something in `outputs/`. As with all of the folders, structuring this further
with some kind of subdirectory structure is encouraged.

There is one special rule involving `workdata/` which helps to clarify its
role: At any point, you must be able to delete the contents of this folder and
lose nothing but computing time (i.e. everything in here must be able to be
generated).


### `scripts/`

This is where the software written for this project goes. The name "scripts"
implies that these are typically small- to medium-sized programs, but nothing
in these rules technically prevents you from putting some industrial Java
monster here, either.

You may never execute anything in here directly (use `make`!), except for
experimentation. Results from direct calls may not be used further.


### `persistent/`

This is human-annotated data that should _not_ be deleted. Examples for files
that belong here are manually annotated parts of the input data (ideally not
reproducing it but referencing it via IDs or similar). Programs do not belong
here, nor do texts meant for humans (LaTeX, readmes, etc.).


### `writing/`

If you are writing an academic paper about this project, this is the place for
your source files, typically `.tex` and `.bib`.


### `Makefile`

In the `Makefile`, you describe how your files in `workdata/` and `outputs/`
are produced, and specify their dependencies. Usually, you'll want a
parameter-less call of `make` to reproduce the entire work, but in case some of
the computation is very intensive (e.g. long training phases), you can describe
several distinct subgraphs and invoke the parts with e.g. `make prepare`, `make
train`, `make results`.

It is recommended to provide common phony targets such as `make all` (that
really reproduces everything), `make clean` (that deletes everything in
`workdata/`), and `make setup`, if needed (see the section on `sources/`).

In principle, instead of `make` you could use some kind of replacement, but
this is not recommended. When handling the `Makefile` gets too complicated,
instead generate it via something like a `configure` script.


### Other files and folders

This is not an exhaustive list of files and folders, although the majority of
the rest should be meta-files, such as:

- READMEs and other forms of documentation
- CI/CD configuration files in case you do that (good!)
- Files describing the programming environment (e.g. a virtualenv and
  `requirements.txt` for Python)
- Version control metadata (`.git/`, `.gitignore`, etc.)
- ...


## Rules

These are the rules of *reproduci*:

0. *reproduci* is meant to adapt to you: None of this is set in stone. If
   something doesn't work for you, change it.
1. Everything in `workdata/` is deletable.
2. Use version control. At the end of a working unit, your working directory
   **must** be clean.
3. Strive for beauty and consistency in structure & naming.
4. The folders may have subfolders in whatever way you feel is helpful.
5. Never execute data-changing commands outside `make`. If you must run a
   command outside `make`, their outputs may not be
   used.
6. Avoid code and data duplication; use symlinks liberally.
7. Use `\input` (or similar) to avoid mixing text and data.
