---
layout: post
title: "AI for Oncology"
data: "2025-02-03 10:00:00 +0000"
categories: til
tags: [AI, oncology, python, DIRECT, SCC]
excerpt: >
  Summary of how the AI for Oncology LAB (NKI, the Netherlands) is planning
  on using AI to automatically segment SCC biopsies intra-operatively, and how they
  developed the DIRECT package to recreate high-quality MRI medical images from under-sampled
  k-space MRI data.
---

## Introcuction

In September of last year I attended a presentation by [Dr. Jonas Teuwen](https://www.nki.nl/research/find-a-researcher/groupleaders/jonas-teuwen/) entitled 'Oncology Foundation Models', where he gave an overview of the [AI for Oncology Lab](https://aiforoncology.nl)'s (Netherlands Cancer Institute, NKI) work on foundational DNN models for reconstructing MRI images from under-sampled k-space MRI measurements. The presentation was inspiring, opening my eyes to a growing field where AI tools were being developed to improve medical practices and patient outcomes. Since then I've kept an eye out for news out of the group and when I saw they had a [vacancy for an AI postdoc](https://aiforoncology.nl/vacancies/phdpostdoc-position-utilizing-artificial-intelligence-to-segment-echo-images-of-tongue-cancer-intraoperatively-to-facilitate-radical-resection/) I thought I'd apply.

The postdoc position will tackle the problem of using intra-operative ultrasound (US) measurements of tongue biopsies to generate a 3D image of the biopsy region and detect (segment) regions exhibiting [squamous cell carcinoma (SCC)](https://en.wikipedia.org/wiki/Squamous-cell_carcinoma). SCC poses a challenge to surgeons as it must be fully removed from a patient, with a 5 mm margin to decrease the chances of remision, but this criteria is unmet in 85% of vases due to limited intra-operative feedback. Currently the biopsy is rapidly stained and sent to a pathologist or radiologist who must provide feedback within 30 minutes. A properly developed AI model will cut down this time significantly, increasing the number of attempts the surgeon can make during the surgery, and ultimitely improve the prognosis for the patient.

## Prior Work

[_Iteroperative resection margin model of tongue carcinoma using 3D reconstructed ultrasound (Bekedam2021)_](https://www.sciencedirect.com/science/article/pii/S2667147621001436): This paper outlines the problem, and the proposes an ultrasound measurement solution without applying AI (a radiologist must still manually segment the specimin intra-operatively).

The SCC should be removed with a margin > 5 mm. Up until the publication of this paper, the only method to assess the margin was by using frozen section analysis (FSA) and post-operative histopathological assessment, which takes days and doen't provide real-time information for surgeons to take action on during the surgery. The paper outlines X alternative techniques for intra-operative feedback:

1. Macroscopic analysis of the specimin by the pathologist and surgeon, but slicing the specimen changes the anatomical orientation, shape and size of the specimin, reducing the quality of any analysis, and requires intensive cooperation between surgeons and the pathologist.
2. The speciminn is assessed locally using a fibre-opitc needle probe and integrated Raman specroscopy.
3. Fluorescence imaging.

The paper goes into detail of how the US images are taken and in the discussion states that the main drawback is the time required for manual segmentation, claiming the entire process takes approximately 33 minutes, of which 25 minutes is manual segmentation. A well calibrated and robust AI segmentation model should cut this segmentation time down to seconds, reducing the feedback time for the surgeon to 8 - 10 minutes.

## Approach to The Problem

[_Learning to Map 2D Ultrasound Images into 3D Space
with Minimal Human Annotation (Yueng2021)_](https://www.sciencedirect.com/science/article/abs/pii/S136184152100044X): This paper presents a solution for mapping 2D US images to 3D images, allowing the user to inspect the 3D images from arbitrary angles and slices that may not have been taken during the US measurement. The authors also provice an extensive [project page](https://pakheiyeung.github.io/PlaneInVol_wp/) with a presentation from the lead author and a github repo [pakheiyeung/PlaneInVol](https://github.com/pakheiyeung/PlaneInVol) with the code.

<div style="text-align: center;">
<p style="margin: 0; text-align: center; font-weight: normal">
    Abstract image from Yueng2021 showing the three stages of the PlaneInVol python package, which takes input 2D US images, runs them through a self-superivsed trained CNN model, and predicts arbitrary slice locations.
    </p>
   <img src="{{ site.baseurl }}/assets/posts/PlaneInVol_abstract_image.jpg" style="width: 75%;">
</div>

[_Detection of COVID-19 features in lung ultrasound images using deep neural networks (Zhao2024)_](https://www.nature.com/articles/s43856-024-00463-5): This paper outlines a U-net model to segment COVID-19 in lung US measurements.

<div style="text-align: center;">
<p style="margin: 0; text-align: center; font-weight: normal">
    Figure from Zhao2024 showing the U-net architecture used to detect COVID-19 in lung US measurements.
    </p>
   <img src="{{ site.baseurl }}/assets/posts/Zhao2024_U-net.png" style="width: 75%;">
</div>

[_Enhancing the reliability of deep learning-based head and neck tumour segmentation using uncertainty estimation with multi-modal images Ren2024_](https://iopscience.iop.org/article/10.1088/1361-6560/ad682d): This paper develops methods to quantifying uncertainty in multi-modal head and neck cancer segmentation models, allowing users to create 3D uncertainty maps that could help medical practicioners focus on high-uncertainty areas.

## Other Work From the Group

Some of the most interesting work out of the group is unrelated to SCC segmentation, but actualy deals with processing under-sampled or noisy k-space MRI data to high-quality images. Models can be used or fine-tuned on specific datasets using the [DIRECT](https://github.com/NKI-AI/direct) python package.

<div style="text-align: center;">
<p style="margin: 0; text-align: center; font-weight: normal">
    Slide taken from <a href="https://www.youtube.com/watch?v=Jl18e2XoHNI" target="_blank">Webinar 33 Accelerated MRI with AI by Jonas Teuwen</a> presented at European Society of Medical Imaging Informatics, showing the principle of how to transform raw, complex k-space MRI data into a mecical image.
    </p>
   <img src="{{ site.baseurl }}/assets/posts/k-space_to_MRI-image.png" style="width: 75%;">
</div>

<div style="text-align: center;">
<p style="margin: 0; text-align: center; font-weight: normal">
    Under-sampled k-space to save time during MRI measurement (decreasing cost, decreasing discomfort for the patient, and reducing artifacts caused by movement over time).
    </p>
   <img src="{{ site.baseurl }}/assets/posts/subsampled-k-space_to_MRI-image.png" style="width: 75%;">
</div>

<div style="text-align: center;">
<p style="margin: 0; text-align: center; font-weight: normal">
    Showing the performance using a "zero-filled" model (no interpolation of missing k-space), CS (Compressed Sensing) model using BART and RIM (recurrent inference machine) using DIRECT.
    </p>
   <img src="{{ site.baseurl }}/assets/posts/direct_models.png" style="width: 75%;">
</div>

DIRECT uses MRI phyiscs as prior knowledge, sampling masks, MRI machine sensitivity masks and a complicated update function (using ReLU convolutional layers, gated recurrent units (GRU)) in a recurrent inference machine (RIM) model to increase the quality of generated MRI images.
