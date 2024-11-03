+++
title = 'GATK HaplotypeCaller Opensource Alternatives'
date = 2024-07-02T20:25:42+08:00
categories = ["Infrasture", "Bioinformatics", "applications"]
draft = true
ShowToc = true
TocOpen = true
+++

## Introduction

It is not easy to modify GATK's code for performance optimizations. GATK strictly followed the Object-Oriented Design(OOD) and most of its classes, objects are strongly coupled with each other. Due to its huge code scale, it's not only hard to comprehend, but also difficult for secondary developments. Thus, to better understand the mechanism of a tool like HaplotypeCaller(HC), it's better to start from some simplifer alterntives.

## gatk-cpp

[Repository](https://github.com/hewillk/gatk-cpp)

A simplfied prototype implemented using C++. Missing a lot of critical details, but still a good project to start.

- Pros: Easy to understand, (Personally)written in C++
- Cons: Missing details, some parts incorrect.

## Freebayes

[Repository](https://github.com/freebayes/freebayes)

A bayesian haplotype-based caller written in C++. It's a validated and continuously-maintained project with rather straightforward code structure. However, its internal logic is not completely identical to GATK-HC.

- Pros: Well-structured, validated, actively-maintained
- Cons: Different main logic, different main algorithms

## Lorikeet

[Repository](https://github.com/rhysnewell/Lorikeet)

A re-implementaion of the GATK HaplotypeCaller written in Rust. It's a personal-maintained project, however well structured. However, I'm not quite familar how it is similar to GATK-HC because I'm not very used to Rust.

- Pros: Well-structured, more identical re-implementaion
- Cons: (Personally)written in Rust

## Conclusion

There are not many alternatives for GATK-HC, after all a re-implementation is a very challenging task. Most current alternatives have their own disadvantages. Thus to better understand GATK-HC, it is better to combine these alternatives and GATK-HC itself to perform our own prototype re-implementation.