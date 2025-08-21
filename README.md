# Aura-Mind: An AI Field Guide for the Nigerian Farmer ✨

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![GitHub release (latest by date)](https://img.shields.io/github/v/release/surfiniaburger/AuraMind)](https://github.com/surfiniaburger/AuraMind/releases)

**https://huggingface.co/spaces/surfiniaburger/Aura-maize**

An offline-first, on-device AI that empowers Nigerian farmers to diagnose maize diseases using just their phone's camera. This project is a submission for the **Google - The Gemma 3n Impact Challenge**.

**Explore Aura-Mind:**
*   **[Kaggle Writeup (Main Story)](https://www.kaggle.com/writeups/surfiniaburger/aura-mind-an-ai-field-guide-for-the-nigerian-farme)** 
*   **[Technical Deep Dive & Training Notebooks](https://github.com/surfiniaburger/tune/blob/main/docs)** 
*   **[Latest APK Release](https://github.com/surfiniaburger/AuraMind/releases/latest)** 
---

## The Problem: A Local Challenge with Global Implications

Agriculture is the backbone of Nigeria's economy, employing over 70% of the population. Yet, farmers face an estimated 30-40% in annual crop losses due to pests and diseases. For a smallholder farmer, this is the difference between prosperity and poverty. The knowledge to fight back exists, but it's inaccessible, especially in rural areas with no reliable internet.

Aura-Mind was born to bridge this gap.

## Our Solution: An AI That Lives on the Edge

Aura-Mind is an AI-powered companion that runs **entirely offline** on a basic Android phone. A farmer can take a picture of a struggling maize plant, and the AI provides a probable diagnosis and simple, actionable advice.

This project fine-tunes Google's powerful **Gemma 3n** model on a custom dataset of local maize conditions, creating a specialized "Maize Expert" that is private, personal, and powerful enough to run in the palm of a farmer's hand.

## Technical Journey: From Debugging Hell to 100% Accuracy

The path to a working model was a trial by fire, battling everything from hardware errors on cloud GPUs to subtle bugs in cutting-edge libraries. Our journey involved:
1.  **Building a Real-World Dataset:** Gathering images from local Lagos markets to ensure our model trained on realistic, not academic, data.
2.  **Establishing a Stable Pipeline:** Through rigorous experimentation, we proved a single-GPU training strategy was superior for model quality.
3.  **Achieving Perfection:** We launched an automated Weights & Biases sweep that successfully identified multiple hyperparameter configurations that achieved **100% validation accuracy**.
4.  **The Final Frontier:** The current challenge is the final conversion of our proven, high-performance model into the `.task` format required for on-device deployment.

For a full, detailed narrative of our technical journey and a deep dive into our training and evaluation methodology, please see our **[accompanying articles](https://github.com/surfiniaburger/tune/blob/main/docs)**.

## The Application

This repository is a fork of the official **Google AI Edge Gallery**, which serves as the production-ready foundation for the Aura-Mind app. We have configured a GitHub Actions CI/CD pipeline that automatically builds and releases a new APK on every commit.

The current `v0.1.0-alpha` release contains the base application. The final step is to integrate our custom-trained `.task` file, which is pending the resolution of the conversion issues detailed in our writeup.

---


