# Working instructions

## Current phase

This repository is in specification and design exploration. Do not implement the launcher, scaffold a framework, install runtime dependencies or create deployment workflows unless the user explicitly asks to move to implementation. The source tree shown in docs/architecture.md is a proposal, not a list of directories to create now.

## Language

Write technical documentation, code identifiers, filenames and development comments in English. Product interfaces target French-speaking young people aged 12-16. Retain the supplied titles: Tik Tak Tik Tak, The Secret of St Christophe, Marked, Without a Trace, Echo, Le groupe parallèle.

## Read before editing

Start with README.md, docs/decisions.md and the relevant canonical contract. docs/api.md owns runtime protocol and display negotiation; docs/experience-integration.md owns catalogue and metadata; docs/architecture.md owns source organisation. Prefer links over duplicate contract definitions.

## Preserve product intent

The launcher has interchangeable responsive themes. A phone is only one optional theme. Both themes and experiences declare supported display modes. The launcher mediates theme controls and experience requests and reports actual state. Navigation visibility is independent from display mode. Mode changes preserve the running iframe. No immersive metadata declaration is needed for navigation hide/show.

## Editing discipline

Distinguish confirmed requirements from proposed implementation details and open questions. Keep related contracts, decisions and acceptance criteria coherent after changes. Record substantive changes in CHANGELOG.md. Do not infer game plots, content categories, release dates or repository URLs from titles. Generated visuals and historical sketches are exploratory, not final design commitments.

Do not commit, push, publish or create a remote repository unless authorised by the user. Do not add AI attribution to commits or pull requests.
