# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a knowledge repository for system design concepts and learning materials. It contains markdown documentation covering distributed systems, scalability patterns, and back-of-envelope calculations.

## Repository Structure

- **GLOSSARY.md** - Central reference for metrics and definitions (availability, latency, QPS, DAU, SLA) with reference tables
- **CONCEPTS/** - Deep-dive explanations of system design concepts (redundancy, powers of two calculations)
- **SETUPS/** - Architecture patterns at different scales (single server, etc.)
- **visuals/** - Diagrams and images referenced by documentation

## Writing Conventions

- Include reference tables for quantitative concepts (availability percentages, latency numbers, powers of two)
- Add worked examples with step-by-step calculations, particularly for back-of-envelope estimations
- Link to authoritative sources (AWS docs, Google Cloud docs, Wikipedia, original papers/talks)
- Cross-reference related documents using relative paths (e.g., `../SETUPS/SINGLE_SERVER_SETUP.md`)
- Healthcare/pharmacy domain is used for worked examples throughout
