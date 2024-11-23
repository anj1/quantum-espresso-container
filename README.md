# Quantum ESPRESSO GPU Container

This repository contains a Dockerfile for building and running Quantum ESPRESSO with GPU support using the NVIDIA HPC SDK. The container provides a complete environment for running quantum chemistry calculations with GPU acceleration.

## Prerequisites

- Docker installed on your system
- NVIDIA Container Toolkit installed
- NVIDIA GPU with CUDA support
- At least 20GB of free disk space

To check if your system is properly configured for running GPU containers:
```bash
docker run --gpus all nvidia/cuda:12.3.0-base-ubuntu22.04 nvidia-smi
```

## Building the Container

1. Clone this repository:
```bash
git clone <repository-url>
cd quantum-espresso-gpu
```

2. Build the Docker image:
```bash
docker build -t qe-gpu
```

This process may take 30-60 minutes depending on your system, as it needs to:
- Download the NVIDIA HPC SDK base image (~10GB)
- Install dependencies
- Compile Quantum ESPRESSO with GPU support

## Usage

### Basic Usage

Run a simple calculation:
```bash
docker run --gpus all -v $(pwd):/work qe-gpu pw.x -input example.in
```

### Advanced Usage

Run with specific GPU and thread configuration:
```bash
docker run --gpus all \
    -e GPU_ID=0 \
    -e OMP_NUM_THREADS=4 \
    -v $(pwd):/work \
    qe-gpu pw.x -input example.in
```

Run an interactive session:
```bash
docker run --gpus all -it \
    -v $(pwd):/work \
    --entrypoint /bin/bash \
    qe-gpu
```

### Environment Variables

- `GPU_ID`: Specify which GPU to use (default: 0)
- `OMP_NUM_THREADS`: Set number of OpenMP threads (default: 1)

## Container Specifications

- Base image: `nvcr.io/nvidia/nvhpc:24.1-devel-cuda12.3-ubuntu22.04`
- Quantum ESPRESSO version: 7.2
- CUDA version: 12.3
- Compute capability: 8.0 (modify `--with-cuda-cc` in Dockerfile for different GPUs)

### Included Components
- pw.x (PWscf)
- cp.x (CP)
- ph.x (PHonon)

## Customization

### Different GPU Architecture

Modify the `--with-cuda-cc` flag in the Dockerfile:

```dockerfile
--with-cuda-cc=80  # For Ampere (RTX 3000, A100)
--with-cuda-cc=86  # For Ampere (RTX 3000 Ti, A6000)
--with-cuda-cc=89  # For Ada Lovelace (RTX 4000)
```

### Different QE Version

Change the `QE_VERSION` environment variable in the Dockerfile:

```dockerfile
ENV QE_VERSION="7.2"  # Change to desired version
```

## Troubleshooting

### Common Issues

1. **Build fails with CUDA error**
   - Verify NVIDIA HPC SDK installation in container
   - Check CUDA compute capability matches your GPU

2. **Runtime GPU not found**
   - Ensure nvidia-container-toolkit is installed
   - Verify docker --gpus flag is working
   - Check GPU is visible with nvidia-smi

3. **Memory errors during large calculations**
   - Add --shm-size flag to docker run:
     ```bash
     docker run --gpus all --shm-size=1g -v $(pwd):/work qe-gpu pw.x -input example.in
     ```

## Performance Tips

1. Match OMP_NUM_THREADS to your CPU cores per GPU
2. Use CPU affinity for better performance:
   ```bash
   docker run --gpus all \
       -e OMP_NUM_THREADS=4 \
       -e OMP_PLACES=cores \
       -e OMP_PROC_BIND=close \
       -v $(pwd):/work \
       qe-gpu pw.x -input example.in
   ```

## Contributing

Feel free to submit issues and enhancement requests!

## License

This container setup is provided under the MIT License. Quantum ESPRESSO is covered by its own license (GPL).

## Citation

If you use this container for your research, please cite:
- P. Giannozzi et al., J. Phys.:Condens. Matter 21 395502 (2009)
- P. Giannozzi et al., J. Phys.:Condens. Matter 29 465901 (2017)
