# FLATS-5G: Federated Learning-Augmented Transformer-LSTM Spectrum Allocation

## Overview
Multi-step traffic prediction and dynamic spectrum allocation for 5G networks using hybrid LSTM-Transformer architecture with federated learning.

## Key Features
- 12-step ahead traffic forecasting (vs. single-step NARNET)
- Hybrid LSTM-Transformer architecture for improved accuracy (target RE < 0.10)
- Privacy-preserving federated learning for multi-operator scenarios
- Dynamic bandwidth allocation with iterative 5MHz assignment
- Comprehensive evaluation against 5 baselines

## Installation

```bash
# Clone repository
git clone https://github.com/YOUR_USERNAME/FLATS-5G-Spectrum-Allocation.git
cd FLATS-5G-Spectrum-Allocation

# Create conda environment
conda create -n flats5g python=3.9
conda activate flats5g

# Install dependencies
pip install -r requirements.txt
