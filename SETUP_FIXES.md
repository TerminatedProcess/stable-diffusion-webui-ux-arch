# Setup Fixes for stable-diffusion-webui-ux (Arch Fork)

This is the UX-improved fork with 430+ additional commits over the original AUTOMATIC1111 webui.

## System Configuration

- **OS**: Garuda Linux (Arch-based)
- **GPU**: NVIDIA RTX 4070 Ti SUPER
- **System CUDA**: 13.0
- **Python**: 3.10.16 (via uv)
- **PyTorch**: 2.1.2+cu118 (CUDA 11.8)

## Required Fixes

### 1. PyTorch CUDA Version

**Issue**: Webui auto-installs PyTorch with cu121 (CUDA 12.1) which causes library version mismatches on CUDA 13.0 system.

**Solution**: Pre-install PyTorch 2.1.2+cu118 before running webui.sh

**Files Modified**: `.salias` install function (line 48)

```fish
python -m pip install torch==2.1.2 torchvision==0.16.2 --index-url https://download.pytorch.org/whl/cu118
```

### 2. Broken Repository URL

**Issue**: `https://github.com/Stability-AI/stablediffusion.git` doesn't exist

**Solution**: Changed to CompVis repository

**Files Modified**: `modules/launch_utils.py`
- Line 349: URL changed to `https://github.com/CompVis/stable-diffusion.git`
- Line 355: Commit hash updated to `69ae4b35e0a0f6ee1af8bb9a5d0016ccb27e36dc`

### 3. Missing taming-transformers Module

**Issue**: `ModuleNotFoundError: No module named 'taming'`

**Solution**:
1. Clone taming-transformers into `repositories/`
2. Create missing `__init__.py`
3. Add to Python path in `modules/paths.py`

**Files Modified**:
- `.salias` install function (lines 50-52)
- `modules/paths.py` line 41 - added taming-transformers to path_dirs
- Created `repositories/taming-transformers/taming/__init__.py`

### 4. Missing ldm.modules.midas Module

**Issue**: `ModuleNotFoundError: No module named 'ldm.modules.midas'`

**Solution**: Created stub module that webui monkey-patches at runtime

**Files Created**:
- `repositories/stable-diffusion-stability-ai/ldm/modules/midas/__init__.py`
- `repositories/stable-diffusion-stability-ai/ldm/modules/midas/api.py`

### 5. Missing ATTENTION_MODES Attribute

**Issue**: `AttributeError: type object 'BasicTransformerBlock' has no attribute 'ATTENTION_MODES'`

**Solution**: Added class attribute to BasicTransformerBlock

**Files Modified**: `repositories/stable-diffusion-stability-ai/ldm/modules/attention.py` lines 197-199

```python
class BasicTransformerBlock(nn.Module):
    ATTENTION_MODES = {
        "softmax": CrossAttention,
    }
```

### 6. Missing ldm.data.util Module

**Issue**: `ModuleNotFoundError: No module named 'ldm.data.util'`

**Solution**: Created stub module with AddMiDaS class

**Files Created**:
- `repositories/stable-diffusion-stability-ai/ldm/data/__init__.py`
- `repositories/stable-diffusion-stability-ai/ldm/data/util.py`

### 7. Missing LatentDepth2ImageDiffusion Class

**Issue**: `ImportError: cannot import name 'LatentDepth2ImageDiffusion'`

**Solution**: Added stub class to ddpm.py

**Files Modified**: `repositories/stable-diffusion-stability-ai/ldm/models/diffusion/ddpm.py` (appended at end)

```python
class LatentDepth2ImageDiffusion(LatentDiffusion):
    """Stub class for depth2img diffusion."""
    pass
```

## Differences from Original AUTOMATIC1111 Fixes

This UX fork required the **same core fixes** as the original:
1. ✅ PyTorch cu118 instead of cu121
2. ✅ CompVis repository URL fix
3. ✅ taming-transformers setup
4. ✅ midas stub modules
5. ✅ ATTENTION_MODES attribute
6. ✅ ldm.data.util module
7. ✅ LatentDepth2ImageDiffusion class

**Note**: The fork maintainers haven't fixed these upstream CompVis repository issues, so the same patches are needed.

## Usage

```bash
# Initial setup
mkenv          # Create Python 3.10.16 venv
install        # Install dependencies and apply fixes

# Run webui
run            # Launches on http://127.0.0.1:7860
```

## Files Modified Summary

1. `.salias` - Installation script with cu118 PyTorch and taming-transformers
2. `modules/launch_utils.py` - Repository URL and commit hash fixes
3. `modules/paths.py` - Added taming-transformers to Python path
4. `repositories/stable-diffusion-stability-ai/` - Multiple stub modules and patches

## Performance

- Webui launches successfully
- Compatible with NVIDIA RTX 4070 Ti SUPER
- CUDA 11.8 PyTorch works with CUDA 13.0 system
- No xformers installed by default (can be added later)
