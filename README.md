# Harmonic Spectra

[![Python](https://img.shields.io/badge/Python-3.9%2B-3776ab?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-009688?style=flat-square&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![SciPy](https://img.shields.io/badge/SciPy-1.10%2B-8caae6?style=flat-square&logo=scipy&logoColor=white)](https://scipy.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](LICENSE)

A small browser studio that draws the frequencies inside a synthesized tone. A Python server builds the wave, runs a Fast Fourier Transform, and the page plots the spectrum, the harmonics, and a spectrogram.

![Harmonic Spectra studio after analyzing a 440 Hz sawtooth](assets/studio-deck.png)

The picture above is the running app, just after it loads. The tone is a **440 Hz sawtooth**, analyzed with a **2048-point Hann window** at **44.1 kHz**. Every number on the deck comes from that response.

On this run the pitch readout is **441.69 Hz (A4, +7 cents)**. That estimate is autocorrelation. The harmonic table is coarser: it reports the loudest FFT bin near each multiple of the pitch. At 2048 points those bins are about **21.5 Hz** apart, so the first harmonic shows up at **430.7 Hz** even though the pitch estimate is 441.69 Hz. Spectral centroid is the center of mass of the whole spectrum. A sawtooth spreads energy across many harmonics, so the centroid here is **6904 Hz**. The 85% rolloff is **1765.7 Hz**. Flatness is **0.0145**, and the page labels that as tonal.

The green bars rise to about **−1.4 dB** for a second after each analysis, then drop. Turn the microphone on and both bars follow that single input level, and the oscilloscope draws the live wave. Pitch, the spectrum, and the harmonic table stay on the last tone the server analyzed.

The corner of the header says "24-bit float." **AUDITION_SIGNAL** plays a 16-bit WAV.

## Same note, three shapes

These plots come from `src/synthesizer.py` and `src/dsp_engine.py`. The spectra use the same 2048-point Hann FFT the studio requests.

![Sine, square, and sawtooth waves at 440 Hz, each with its spectrum](assets/waveform-spectra.png)

A sine wave is one peak at the fundamental. A square wave is dominated by odd harmonics (1f, 3f, 5f, …), with smaller spikes between them left over from the window. A sawtooth keeps even and odd harmonics, so the spectrum stays busy as the frequency climbs. Dots mark harmonics the detector reported above −30 dB.

## A sweep, as a spectrogram

![Spectrogram of the built-in chirp, sweeping from 100 Hz to 4 kHz](assets/chirp-spectrogram.png)

The **SWP** button glides from 100 Hz to 4 kHz. The picture is a 2 second glide from that same generator, using the studio's spectrogram settings: a Hann window, a 1024-point FFT, and a hop of 512 samples. The page draws a downsampled copy of that transform in the waterfall.

A steady sawtooth barely changes over time, so the waterfall in the studio screenshot is a flat band. The sweep is where time shows up. The large spectrum and the harmonic table only see the first FFT window, which on a sweep is the beginning of the glide, near 100 Hz. The pitch tracker is built for a steady note, so on a sweep it can settle on a different frequency. The spectrogram is the view that follows the glide.

## Waveforms

| Button | What gets generated |
| --- | --- |
| SAW | Sawtooth at the slider frequency |
| SQR | Square wave. Odd harmonics dominate. |
| TRI | Triangle wave |
| SIN | Sine wave |
| ORG | Added sines at 1×, 2×, 3×, 4×, and 8× the slider frequency |
| SWP | Logarithmic sweep from 100 Hz to 4 kHz |

The pitch slider runs from 55 Hz to 1760 Hz. FFT size is 1024, 2048, or 4096 points. Window choices are Hann, Hamming, Blackman, and rectangular.

## Run it

From this folder, with Python 3.9 or newer:

```bash
pip install -r requirements.txt
uvicorn src.app:app --host 127.0.0.1 --port 8001 --reload
```

Open [http://localhost:8001](http://localhost:8001).

### Tests

```bash
pytest -v tests/
```

The tests check waveform generation, the WAV header, pitch on a 440 Hz sine, harmonic extraction, spectral flatness, and the health and analyze-synthetic routes.

## API

| Method | Path | Response |
| --- | --- | --- |
| `GET` | `/api/health` | Status, name, and version |
| `POST` | `/api/analyze-synthetic` | Generated samples plus FFT (pitch, centroid, rolloff, flatness, harmonics, spectrum), a spectrogram grid, oscilloscope samples, and a base64 WAV |
| `POST` | `/api/analyze-raw` | FFT of a sample array you send |

## License

Released under the MIT License. See [LICENSE](LICENSE).
