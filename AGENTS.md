# AGENTS.md - Whisper-WebUI

> Guidelines for AI coding agents (Jules AI, GitHub Copilot, Codex, etc.) working on this repository.

## Project Overview

Whisper-WebUI is a Gradio-based browser interface for OpenAI Whisper speech recognition. It supports multiple Whisper implementations, translation, VAD, speaker diarization, and background music separation.

### Architecture

```
Whisper-WebUI/
├── app.py                      # Main Gradio application entry point
├── requirements.txt            # Python dependencies (CUDA 12.6 default)
├── modules/
│   ├── whisper/                # Whisper implementations
│   │   ├── base_transcription_pipeline.py  # Abstract base class
│   │   ├── whisper_factory.py              # Factory pattern for implementations
│   │   ├── faster_whisper_inference.py     # Faster-Whisper (default)
│   │   ├── whisper_Inference.py            # OpenAI Whisper
│   │   ├── insanely_fast_whisper_inference.py  # Insanely Fast Whisper
│   │   └── data_classes.py                 # Pydantic models & enums
│   ├── translation/            # NLLB & DeepL translation
│   ├── vad/                    # Silero Voice Activity Detection
│   ├── diarize/                # Speaker diarization (pyannote)
│   ├── uvr/                    # Background music separation
│   └── utils/                  # Utilities (paths, logger, audio, etc.)
├── backend/                    # FastAPI REST API
│   ├── main.py                 # API entry point
│   ├── routers/                # API endpoints
│   └── configs/                # Backend config & .env
├── configs/
│   ├── default_parameters.yaml # Default Whisper/VAD/Diarization params
│   └── translation.yaml        # i18n strings
├── tests/                      # Test suite
└── models/                     # Downloaded model storage
```

---

## Environment Setup

### Prerequisites
- **Python**: 3.10 - 3.12
- **FFmpeg**: Must be in system PATH
- **CUDA**: 12.6+ recommended (or edit `requirements.txt` for other versions)
- **GPU Memory**: 4GB+ VRAM recommended

### Installation Commands

```bash
# Clone repository
git clone https://github.com/jhj0517/Whisper-WebUI.git
cd Whisper-WebUI

# Create virtual environment
python -m venv venv

# Activate venv
# Linux/Mac:
source venv/bin/activate
# Windows:
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### CUDA Configuration
Edit `requirements.txt` first line for your CUDA version:
```
--extra-index-url https://download.pytorch.org/whl/cu126  # CUDA 12.6
--extra-index-url https://download.pytorch.org/whl/cu128  # CUDA 12.8
--extra-index-url https://download.pytorch.org/whl/xpu    # Intel GPU
```

---

## Running the Application

```bash
# Start WebUI (default: http://localhost:7860)
python app.py

# With custom options
python app.py --whisper_type faster-whisper --server_port 7860 --share

# Start Backend API (default: http://localhost:8000)
uvicorn backend.main:app --host 0.0.0.0 --port 8000
```

---

## Testing

### Test Commands (Jules AI: Run these to validate changes)

```bash
# Install test dependencies
pip install pytest

# Run ALL tests
pytest tests/ -v

# Run specific test modules
pytest tests/test_transcription.py -v      # Core transcription tests
pytest tests/test_translation.py -v        # Translation tests
pytest tests/test_vad.py -v                # VAD tests
pytest tests/test_diarization.py -v        # Speaker diarization tests
pytest tests/test_bgm_separation.py -v     # BGM separation tests

# Run backend tests
pytest backend/tests/ -v

# Run with coverage
pytest tests/ --cov=modules --cov-report=html
```

### Test Configuration
Tests use `tests/test_config.py` for configuration:
- `TEST_WHISPER_MODEL`: Model size for tests (default: small model for speed)
- `TEST_FILE_PATH`: Path to test audio file
- `TEST_ANSWER`: Expected transcription output

### Writing New Tests
Follow the pattern in existing tests:
```python
import pytest
from modules.whisper.whisper_factory import WhisperFactory
from modules.whisper.data_classes import *

@pytest.mark.parametrize("whisper_type", [
    WhisperImpl.FASTER_WHISPER.value,
    WhisperImpl.WHISPER.value,
])
def test_your_feature(whisper_type):
    whisper_inf = WhisperFactory.create_whisper_inference(whisper_type=whisper_type)
    # Your test logic here
    assert result == expected
```

---

## Key Implementation Patterns

### Adding a New Whisper Implementation

1. **Create new inference class** in `modules/whisper/`:
   - Inherit from `BaseTranscriptionPipeline`
   - Implement abstract methods: `transcribe()`, `update_model()`

2. **Register in factory** (`modules/whisper/whisper_factory.py`):
   ```python
   elif whisper_type == WhisperImpl.YOUR_IMPL.value:
       return YourWhisperInference(...)
   ```

3. **Add enum value** (`modules/whisper/data_classes.py`):
   ```python
   class WhisperImpl(Enum):
       YOUR_IMPL = "your-impl"
   ```

4. **Add CLI argument** in `app.py`

5. **Write tests** in `tests/test_transcription.py`

### Data Flow
```
Audio Input → VAD (optional) → BGM Separation (optional) → Whisper Transcription → Diarization (optional) → Output
```

---

## Coding Conventions

- **Type Hints**: Always use Python type hints
- **Logging**: Use `modules/utils/logger.py` - `get_logger()`
- **Config**: Load from YAML in `configs/`
- **Models**: Pydantic `BaseModel` for data classes
- **Factory Pattern**: Use for creating implementations
- **Abstract Base**: Inherit from base classes in `modules/*/`

---

## File Modification Guidelines

### Safe to Modify
- `modules/whisper/*.py` - Whisper implementations
- `modules/*/` - Feature modules
- `tests/*.py` - Test files
- `configs/*.yaml` - Configuration files

### Modify with Caution
- `requirements.txt` - Update `--extra-index-url` comments when changing
- `app.py` - Main entry point, affects all UI
- `backend/main.py` - API entry point

### Do NOT Modify
- `models/` - Auto-generated model storage
- `outputs/` - Auto-generated outputs
- `.github/` - CI/CD workflows (unless specifically asked)

---

## Common Tasks for AI Agents

### Task: Add New Feature
1. Identify the appropriate module in `modules/`
2. Follow existing patterns in that module
3. Add tests in `tests/`
4. Update `configs/default_parameters.yaml` if needed
5. Run `pytest tests/ -v` to validate

### Task: Fix Bug
1. Reproduce with existing tests or write new test
2. Locate issue in `modules/`
3. Fix and run relevant tests
4. Run full test suite before committing

### Task: Add New Whisper Backend
1. Create `modules/whisper/new_impl_inference.py`
2. Inherit `BaseTranscriptionPipeline`
3. Add to `WhisperImpl` enum
4. Update `WhisperFactory`
5. Add tests with `@pytest.mark.parametrize`

---

## Quick Reference

| Action | Command |
|--------|---------|
| Run WebUI | `python app.py` |
| Run API | `uvicorn backend.main:app --port 8000` |
| Run all tests | `pytest tests/ -v` |
| Run single test | `pytest tests/test_transcription.py -v` |
| Check types | `mypy modules/` |
| Format code | `black modules/ tests/` |

---

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `HF_TOKEN` | HuggingFace token for pyannote diarization | For diarization |
| `DEEPL_API_KEY` | DeepL API key for translation | For DeepL translation |
| `DB_URL` | Database URL for backend | For backend API |

---

*This file helps AI agents understand the codebase structure and development workflow. Update when adding new modules or changing architecture.*
