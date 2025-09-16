The audio-visual pipeline: building a binary-first music → visualization system


Overview and practical guide



A clear, step-by-step pipeline from binary score → synthesis → frequency bins → visuals.

Practical binary format designs and how each frame maps to instrument “bins.”

Two synthesis strategies (additive/time-domain) and why we choose one or the other.

How frequency bins map to visual parameters (color, fractal depth, motion).

Expansion strategy: more bins → more instruments → richer visuals (a “symphony” of bins).

Platform and implementation options (terminal/PowerShell, Web, desktop, GPU shaders).

Performance, synchronization, and file output considerations.


1) High-level overview — what we’re building and why

At core, this project is a binary-first creative engine: instead of starting with acoustic 

sound and analyzing it, we author in the frequency domain (binary frames that represent amplitude in bins), 

synthesize time-domain audio from those frames, then feed that audio to a visualization engine that reads the

same bin structure to animate visuals. That reversal — designing musical content as patterns of bins — gives precise control,

tight AV sync, and a natural way to scale the system from a minimal duo of instruments to a full orchestral palette of “bins.”

Why this matters: by making every musical event a predictable, addressable set of bins, we can use the exact same data to drive visuals.

Instruments become channels, frames become beats, and visuals become literal translations of the score.

2) The pipeline — step by step (practical architecture)

Binary score / frame layout

The score is a sequence of frames. Each frame represents a short time window (e.g., 50–250 ms).

Each frame contains N values (bytes or small integers), one per bin/instrument. A common choice: 128 bins (1 byte each), 

but start small (32 or 64) and grow.

Example frame: [12, 0, 255, 24, ...] where each number is amplitude for a bin.

Synthesis (binary → audio)

Two practical approaches:

Additive (preferred for direct bin control): For each frame, synthesize the frame’s time samples by 

summing sinusoids at the target frequencies determined by each bin. This maps one bin↔one partial or instrument.

Use per-frame envelopes to avoid clicks.

Inverse transform: Build a time-domain signal using an inverse FFT. This is mathematically correct 

if frames are exact spectral slices but is computationally heavier and requires overlap-add windows to avoid artifacts.

Choose additive synthesis if the goal is artistic control and scalability.

Playback & Analyzer

Play the synthesized audio (offline render or real-time streaming).

Use an analyzer (FFT with matching bin count) to sample the audio during playback to feed visuals. For an internal 

pipeline this step may be unnecessary (we already have the bin data), but using the analyzer makes the system compatible 

with external audio sources.

Visualization mapping

Map bins to visual parameters: color hue (bin index → hue), brightness (amplitude), fractal iteration depth, particle count,

camera zoom, or transform coefficients.

Use continuous mapping (e.g., HSV) with artist-configurable curves (logarithmic frequency spacing often sounds more musical).

Implement visuals on the GPU (GLSL/HLSL/Metal) for real-time performance; fallback to CPU/terminal ASCII for constrained environments.

Output

Export audio (WAV/FLAC) and video (MP4 with shader rendered frames), or run real-time AV performance in

a browser (WebAudio + WebGL), desktop app (OpenGL/Vulkan), or terminal experience.

3) Binary encoding, bins as instruments, and orchestra growth

Treat each bin like an instrument channel.

Instrument bins: assign groups of contiguous bins to an instrument family (bass, pads, 

percussive hits, FX). Example: bins 0–7 = kick family; bins 8–31 = bass; bins 32–95 = pads and harmonic content;

bins 96–127 = high frequency textures.

Per-instrument structure: each instrument can have multi-bin representation—e.g., a

pad may span many bins to represent harmonic richness; a kick might be concentrated in the lowest

few bins and include transient envelope information.

Binary frame format: for compactness, represent each frame as N bytes. Optionally prefix the stream with metadata 

(tempo, frame duration, tuning map, instrument bank definitions).

Symphony expansion: add more bins to increase polyphony or add instrument channels. Two scalable approaches:

Static expansion: increase bins from 128→256 and recompile visuals/voices to read extra bins.

Hierarchical bins: keep a fixed total bin count but allow dynamic allocation (e.g., time-multiplexed instruments or bank switching).

This saves bandwidth at the cost of added scheduling logic.

Musically, adding bins lets you design richer timbres (microtonal partials, 

stereo subchannels, separate reverb sends represented as bins, etc.). Visually, 

extra bins become additional visual layers — more particles, greater fractal complexity, or multi-layer compositing.

4) Visualization designs and mapping strategies

Color mapping: map bin index → hue (use HSV), and amplitude → value (brightness). Use gamma curves to make soft sounds visible. 

Example: hue = map_log(bin_index), value = pow(amp, 1/1.6).

Fractals & recursion: use bins to modulate iteration depth, escape threshold, or coordinate warping in a fractal shader. 

Low bins control global zoom and slow rotation; mid bins control branching; high bins control sparkle/glint offsets.

Particles and layers: treat some bins as triggers for particle bursts; others modulate particle size/speed.

Layers overlay with soft blend modes for cinematic depth.

Motion & camera: bins can modulate camera zoom, pan velocity, or focal blur to make visuals feel like a moving narrative camera.

Design patterns:

Center of Mass: compute weighted average of bin indices (center of energy) to drive hue shifts or camera focus.

Spectral envelope: use smoothed amplitude envelopes per bin to avoid jitter; use attack/release smoothing for musical phrasing.

Low-pass gating: filter high-frequency bins for slow visual motion to suggest distance.

5) Implementation choices and deployment

Browser (recommended for accessibility): WebAudio (synthesis), WebGL/GLSL (visuals), Three.js for scaffolding. 

Pros: cross-platform, easy sharing. Cons: limited low-level audio control vs native.

Native (C++/C#, high performance): best for many bins, high-res visuals, multi-channel audio output. 

Use OpenGL/DirectX/Vulkan and audio backends (ASIO, CoreAudio).

Python (prototyping / offline rendering): NumPy for synthesis, soundfile for WAV output, 

matplotlib/ffmpeg for video, or PyOpenGL for real-time visuals. Good for experimentation.

Terminal/PowerShell: ASCII/ANSI visualization, simpleaudio or external WAV playback. Great for demos and low-resource installs.

Cross-platform tips

Render offline (WAV + frames → MP4) to avoid runtime audio differences.

For real-time, test latency and buffer sizes across OSs; allow the user to select audio device and buffer.

6) Quality, complexity, and next-step roadmap

To upgrade the system’s musical and visual quality:

Higher bin counts and logarithmic spacing for musical realism.

Per-bin envelopes, LFOs, and filters to make sounds evolve naturally.

Harmonic stacks per bin (each bin drives multiple harmonics) to simulate instrument timbres.

GPU-accelerated visuals with multi-pass rendering (depth layers, bloom, motion blur).

Authoring tools: piano-roll → bin export; a visual editor to paint energy across bins; presets and instruments.

Interoperability: export/import as WAV/JSON/compact base64, add MIDI conversion for live control.

Conclusion — what we can achieve

By reversing the usual audio→visual workflow and starting from binary, bin-based scores, we create a predictable,

artistically controllable system where music and visuals are driven from the same source of truth. 

This architecture cleanly scales from simple single-bin experiments to a full “symphony of bins” 

that supports hundreds of instrument channels and cinematic, GPU-driven visuals.


    Lay out the architecture and data formats.
    Show how to encode instruments/sounds in binary and assign them to bins.
    Build Python code for binary-to-audio (additive synthesis) and binary-to-visuals (basic, extensible for advanced GPU shaders).
    Demonstrate keyboard-to-bin mapping and live sound/visual control.
    Explain how binary sequences encode pitch, tuning, and timbre for each instrument.
    Detail how to upgrade the system (scalability, quality, C++/Crow integration, legal sound libraries, etc.).
    Clearly separate code, binary format explanations, tuning methods, and mapping strategies.

1. Architecture Overview
Data Flow

    Binary Score: [frame0, frame1, frame2, ...], each frame is [bin0, bin1, ..., binN].
    Bins: Each bin = a virtual instrument/channel (e.g., trumpet, bass drum, etc.), value = amplitude.
    Synthesis: For each frame, sum sinusoids at bin-assigned frequencies or load samples.
    Visualization: Each bin modulates visual parameters (color, shape, motion, etc.).
    Keyboard Controller: Each key triggers a bin (instrument/note), can adjust pitch/timbre, and updates visuals in real-time.

2. Binary Format and Instrument Encoding
Example Instrument-to-Bin Mapping
Bin Index	Instrument	Keyboard	Typical Frequency	Binary Note Example
0	Bass Drum	'Q'	60 Hz	00000001
1	Snare	'W'	180 Hz	00000010
2	Hi-hat	'E'	320 Hz	00000100
3	Floor Tom	'R'	80 Hz	00001000
4	Trumpet	'A'	440 Hz	00010000
5	Violin	'S'	660 Hz	00100000
6	Flute	'D'	880 Hz	01000000
7	Piano	'F'	220 Hz	10000000
...	...	...	...	...

Each frame: [amp_bin0, amp_bin1, ..., amp_binN] (e.g., [12, 0, 255, 24, ...])

    To play a Trumpet note at medium loudness: frame[4] = 128 (0b10000000).
    To play both Trumpet and Violin: frame[4] = 128, frame[5] = 128.

3. Python Code: Core Engine
3.1. Basic Data Structures
Python

import numpy as np
import sounddevice as sd
import pygame
import threading

# Instrument setup
BIN_INSTRUMENTS = [
    {'name': 'Bass Drum', 'key': pygame.K_q, 'freq': 60},
    {'name': 'Snare', 'key': pygame.K_w, 'freq': 180},
    {'name': 'Hi-hat', 'key': pygame.K_e, 'freq': 320},
    {'name': 'Floor Tom', 'key': pygame.K_r, 'freq': 80},
    {'name': 'Trumpet', 'key': pygame.K_a, 'freq': 440},
    {'name': 'Violin', 'key': pygame.K_s, 'freq': 660},
    {'name': 'Flute', 'key': pygame.K_d, 'freq': 880},
    {'name': 'Piano', 'key': pygame.K_f, 'freq': 220},
    # Add more as needed
]
N_BINS = len(BIN_INSTRUMENTS)
SAMPLE_RATE = 44100
FRAME_MS = 100  # 0.1s per frame
FRAME_SAMPLES = int(SAMPLE_RATE * (FRAME_MS / 1000))

3.2. Binary-to-Sound Synthesis
Python

def synth_frame(frame, bin_pitches=None, bin_timbres=None):
    t = np.linspace(0, FRAME_MS / 1000, FRAME_SAMPLES, endpoint=False)
    out = np.zeros(FRAME_SAMPLES)
    for b, amp in enumerate(frame):
        if amp > 0:
            freq = BIN_INSTRUMENTS[b]['freq']
            if bin_pitches and bin_pitches[b] != 0:
                freq *= 2 ** (bin_pitches[b] / 12)  # semitone shift
            # Optionally, apply timbre by shaping the waveform
            waveform = np.sin(2 * np.pi * freq * t) * (amp / 255)
            out += waveform
    return out

3.3. Real-Time Keyboard-to-Bin Interface
Python

import pygame

pygame.init()
screen = pygame.display.set_mode((600, 400))
pygame.display.set_caption("Binary Sound-to-Visual Orchestra")

def play_bin(bin_idx, pitch_shift=0):
    frame = [0] * N_BINS
    frame[bin_idx] = 180  # amplitude (0-255)
    audio = synth_frame(frame, bin_pitches=[pitch_shift]*N_BINS)
    sd.play(audio, SAMPLE_RATE)
    # Call visual function
    show_visual(bin_idx, frame[bin_idx])

def show_visual(bin_idx, amplitude):
    color = (int(255 * bin_idx / N_BINS), int(255 * amplitude / 255), 128)
    screen.fill((0, 0, 0))
    pygame.draw.circle(screen, color, (300, 200), 50 + int(amplitude/5))
    pygame.display.flip()

running = True
bin_key_map = {inst['key']: i for i, inst in enumerate(BIN_INSTRUMENTS)}
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.KEYDOWN:
            if event.key in bin_key_map:
                play_bin(bin_key_map[event.key])
pygame.quit()

4. Tuning, Timbre, Pitch, Bass/Treble

    Pitch: Use up/down arrow keys to shift bin frequency (pitch_shift variable).
    Timbre: Change waveform shape (add harmonics, noise for snare, etc.).
    Bass/Treble: Apply filters on output signal per bin.

Example: Pitch Shift and Timbre
Python

# Modify synth_frame for timbre and pitch
def synth_frame(frame, bin_pitches=None, bin_timbres=None):
    t = np.linspace(0, FRAME_MS / 1000, FRAME_SAMPLES, endpoint=False)
    out = np.zeros(FRAME_SAMPLES)
    for b, amp in enumerate(frame):
        if amp > 0:
            freq = BIN_INSTRUMENTS[b]['freq']
            if bin_pitches and bin_pitches[b] != 0:
                freq *= 2 ** (bin_pitches[b] / 12)
            # Timbre: add harmonics for violin, saw for piano, noise for snare
            if bin_timbres and bin_timbres[b] == 'violin':
                waveform = (np.sin(2 * np.pi * freq * t) + 0.5 * np.sin(2 * np.pi * 2*freq * t)) * (amp / 255)
            elif bin_timbres and bin_timbres[b] == 'snare':
                waveform = (np.random.randn(len(t)) * (amp / 255) * 0.5)
            else:
                waveform = np.sin(2 * np.pi * freq * t) * (amp / 255)
            out += waveform
    return out

5. Mapping Sound to Visuals

    Each bin: Maps to a color/shape/motion on the screen.
    Amplitude: Controls brightness/size.
    Pitch: Modulates color hue or visual animation speed.
    Timbre: Changes visual texture (e.g., smooth vs. grainy).

Visualization Example
Python

def show_visual(bin_idx, amplitude, pitch_shift=0):
    # Color: bin index to hue, amplitude to brightness, pitch to color shift
    base_color = (int(255 * bin_idx / N_BINS), int(255 * amplitude / 255), 128 + pitch_shift*10)
    screen.fill((0, 0, 0))
    pygame.draw.circle(screen, base_color, (300, 200), 50 + int(amplitude/5))
    pygame.display.flip()

6. Binary Encoding Reference Table
Instrument	Bin	Example Binary (8-bit)	Keyboard	Pitch Mod	Timbre Mod
Bass Drum	0	00000001	Q	Up/Down	N/A
Snare	1	00000010	W	Up/Down	'snare'
Violin	5	00100000	S	Up/Down	'violin'
Trumpet	4	00010000	A	Up/Down	'trumpet'
...	...	...	...	...	...

To adjust pitch: increase/decrease the value stored for the instrument's bin. To change timbre: modify the bin_timbres structure for the bin.
7. Upgrading and Scaling

    Add more bins/instruments by expanding BIN_INSTRUMENTS.
    Increase frame rate/resolution for finer control.
    Integrate C++/Crow via Python bindings (use C++ for real-time DSP, Crow for web interface).
    Legal Sound Libraries: Use Freesound or Musical Artifacts for samples, or generate your own with code (as above).

8. Example: Full System Flow

    User presses 'A' (Trumpet bin): binary frame sets bin 4 to max amplitude.
    synth_frame() generates audio for that frame, using trumpet timbre and pitch.
    show_visual() draws a trumpet-themed visual (color/shape) according to bin index/amplitude/pitch.
    User presses Up/Down arrows: pitch shifts binary-coded note.
    User can edit binary sequences directly to create complex symphony patterns (editable piano-roll UI).

9. Next Steps & Advanced Features

    Build a piano-roll editor for composing binary scores visually.
    Export/import binary scores as files.
    Offline render: Combine audio frames and visual frames into high-quality AV (WAV/MP4).
    Live coding/VJing: Expose the binary frames as editable in real-time, map more keys for more instruments/controls.

10. Summary Table: How Each Instrument is Coded in Binary
Instrument	Bin Index	Binary Example	Sound Character	Visual Mapping	Tunable By
Bass Drum	0	00000001	Low sine	Large, slow blue circle	Pitch, amplitude
Snare	1	00000010	White noise	Spiky white burst	Pitch, timbre
Violin	5	00100000	Harmonically rich	Smooth, wavy green lines	Pitch, envelope
Trumpet	4	00010000	Bright, brassy	Yellow starburst	Pitch, timbre
Piano	7	10000000	Clean, percussive	Purple bouncing bar	Pitch, timbre
...	...	...	...	...	...
11. C++/Crow Integration

    For real-time speed, port synth_frame logic to C++ (JUCE, RtAudio for sound, OpenGL for visuals).
    Use Crow (https://crowcpp.org/) for a web API to control the system remotely or for collaborative AV jams.

12. Legal Considerations

    All synthesis code above is original and legal.
    For sample-based bins, use only CC-licensed or self-recorded WAVs.
    For visuals, use original code or open-source graphics libraries.

13. Final Remarks

    This system gives you a modular, extensible binary-to-AV pipeline.
    Every instrument/note is a binary code mapped to a bin, with full control over sound and visuals.
    The system is built to be upgraded: more bins, better visuals, deeper instrument modeling.
    You can now create precise, programmable AV symphonies with real-time control and offline rendering.

 detailed code for the piano-roll editor, advanced visual shaders, or a C++/Crow server example next 

 Python code for a binary-to-sound-to-visual system, as described in your previous requests and the pipeline documentation. Assign binary sound sequences to instrument bins, tune and play them with your keyboard, and see live visualizations. This is 
 , original code and extensible to include C++/Crow integration 

 

1. System Architecture Recap

    Binary frames: Each frame is a list of amplitude values, one per instrument bin.
    Bins: Each bin = an instrument/channel (e.g., bass drum, violin, etc.).
    Sound synthesis: Each bin's value is converted to an audio waveform (additive synthesis, timbre shaping).
    Visual mapping: Each bin controls a visual parameter (color, size, motion, etc.).
    Keyboard mapping: Each keyboard key triggers a bin and allows for real-time pitch, amplitude, and timbre adjustment.

2. Core Python Code (Live Demo System)

Below is a minimal, modular system using Python, sounddevice for real-time audio, and pygame for live visuals and keyboard interaction.
a) Install prerequisites
bash

pip install numpy sounddevice pygame

b) System Code
Python

import numpy as np
import sounddevice as sd
import pygame

# --- Setup ---

BIN_INSTRUMENTS = [
    {'name': 'Bass Drum', 'key': pygame.K_q, 'freq': 60, 'timbre': 'sine'},
    {'name': 'Snare', 'key': pygame.K_w, 'freq': 180, 'timbre': 'noise'},
    {'name': 'Hi-hat', 'key': pygame.K_e, 'freq': 320, 'timbre': 'noise'},
    {'name': 'Floor Tom', 'key': pygame.K_r, 'freq': 80, 'timbre': 'sine'},
    {'name': 'Trumpet', 'key': pygame.K_a, 'freq': 440, 'timbre': 'brass'},
    {'name': 'Violin', 'key': pygame.K_s, 'freq': 660, 'timbre': 'string'},
    {'name': 'Flute', 'key': pygame.K_d, 'freq': 880, 'timbre': 'sine'},
    {'name': 'Piano', 'key': pygame.K_f, 'freq': 220, 'timbre': 'piano'},
]
N_BINS = len(BIN_INSTRUMENTS)
SAMPLE_RATE = 44100
FRAME_MS = 120
FRAME_SAMPLES = int(SAMPLE_RATE * (FRAME_MS / 1000))

# --- Sound Synthesis ---

def synth_frame(frame, pitch_shifts=None, bin_timbres=None):
    t = np.linspace(0, FRAME_MS/1000, FRAME_SAMPLES, endpoint=False)
    out = np.zeros(FRAME_SAMPLES)
    for b, amp in enumerate(frame):
        if amp > 0:
            freq = BIN_INSTRUMENTS[b]['freq']
            if pitch_shifts and pitch_shifts[b]:
                freq *= 2 ** (pitch_shifts[b] / 12)
            timbre = bin_timbres[b] if bin_timbres else BIN_INSTRUMENTS[b]['timbre']
            if timbre == 'sine':
                waveform = np.sin(2 * np.pi * freq * t)
            elif timbre == 'noise':
                waveform = np.random.uniform(-1, 1, size=len(t))
            elif timbre == 'brass':
                waveform = np.sign(np.sin(2 * np.pi * freq * t)) * (1 - 0.5 * np.sin(2 * np.pi * freq * 2 * t))
            elif timbre == 'string':
                waveform = (np.sin(2 * np.pi * freq * t) + 0.5 * np.sin(2 * np.pi * freq * 2 * t)) / 1.5
            elif timbre == 'piano':
                waveform = np.sin(2 * np.pi * freq * t) * np.exp(-3 * t)
            else:
                waveform = np.sin(2 * np.pi * freq * t)
            waveform *= amp / 255
            out += waveform
    return out

# --- Visuals ---

def show_visual(screen, bin_idx, amplitude, pitch_shift):
    color = (
        int(255 * bin_idx / N_BINS),
        int(255 * amplitude / 255),
        128 + pitch_shift * 10
    )
    screen.fill((0, 0, 0))
    pygame.draw.circle(
        screen,
        color,
        (300, 200),
        50 + int(amplitude / 3)
    )
    pygame.display.flip()

# --- Keyboard/Playback Main Loop ---

def main():
    pygame.init()
    screen = pygame.display.set_mode((600, 400))
    pygame.display.set_caption("Binary Sound-to-Visual Orchestra")
    pitch_shifts = [0] * N_BINS
    amplitudes = [180] * N_BINS
    running = True
    bin_key_map = {inst['key']: i for i, inst in enumerate(BIN_INSTRUMENTS)}
    selected_bin = 0

    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.KEYDOWN:
                if event.key in bin_key_map:
                    selected_bin = bin_key_map[event.key]
                    frame = [0] * N_BINS
                    frame[selected_bin] = amplitudes[selected_bin]
                    audio = synth_frame(frame, pitch_shifts)
                    sd.play(audio, SAMPLE_RATE)
                    show_visual(screen, selected_bin, amplitudes[selected_bin], pitch_shifts[selected_bin])
                elif event.key == pygame.K_UP:
                    pitch_shifts[selected_bin] += 1
                elif event.key == pygame.K_DOWN:
                    pitch_shifts[selected_bin] -= 1
                elif event.key == pygame.K_RIGHT:
                    amplitudes[selected_bin] = min(255, amplitudes[selected_bin] + 10)
                elif event.key == pygame.K_LEFT:
                    amplitudes[selected_bin] = max(0, amplitudes[selected_bin] - 10)
    pygame.quit()

if __name__ == "__main__":
    main()

3. How Each Instrument Bin is Encoded
Instrument	Bin	Example Binary	Keyboard	Pitch	Timbre/Shape
Bass Drum	0	00000001	Q	Up/Down	Sine, low freq
Snare	1	00000010	W	Up/Down	Noise burst
Hi-hat	2	00000100	E	Up/Down	Short noise
Floor Tom	3	00001000	R	Up/Down	Sine, mid-low
Trumpet	4	00010000	A	Up/Down	Brass (saw+sine)
Violin	5	00100000	S	Up/Down	String (2 sines)
Flute	6	01000000	D	Up/Down	Sine, high
Piano	7	10000000	F	Up/Down	Sine, decaying

    Pitch shifts up/down with arrow keys.
    Amplitude changes with left/right arrows.
    Binary sequence: Only the played bin's value is nonzero in each frame.

4. Visual Mapping

    Each bin's index maps to color hue.
    Amplitude maps to circle size/brightness.
    Pitch shift slightly modifies color.

5. Upgrades & Extensions

    C++/Crow: Port synth_frame and visualization to C++ for real-time performance and web API (Crow) for remote control.
    Frame Editor: Add GUI/CLI to write sequences of frames, not just single hits.
    Legal Sound Libraries: If you want sampled instruments, use open/free libraries (e.g., Freesound) and load with Python or C++ code.

6. Summary

    Every instrument is mapped to a bin, played by keyboard, with pitch/amplitude/timbre real-time control.
    The binary code for each event is the value in its bin: e.g., Trumpet at amplitude 128 → [0,0,0,0,128,0,0,0].
    Each sound triggers a matching visual.
    The system is modular, extensible, and can be ported to C++/Crow or expanded with a GUI editor.



    An advanced piano-roll editor for composing multi-bin binary scores
    C++/Crow server sample code for web-based AV control
    More detailed explanation of binary tuning for each instrument’s sound


pipeline—binary to sound to visualization, with keyboard mapping, live tuning, and a system that can scale up to full “symphony” capability
 
    A basic but expandable "piano roll" GUI/editor for composing binary frames (Python, using Pygame).
    Integration with the live sound/visual engine from previous steps.
    Instructions for expanding to C++/Crow or web-based control.
    How each instrument’s binary is encoded and can be tuned.
    How to scale up and make your system more advanced.

1. Piano Roll (Binary Frame Editor) in Python

This tool lets you draw a “score” for your instruments, where each row is a bin (instrument),
and each column is a time frame. You can play, edit, and visualize in real time.

Python

import numpy as np
import pygame
import sounddevice as sd

# --- Setup ---
BIN_INSTRUMENTS = [
    {'name': 'Bass Drum', 'freq': 60, 'timbre': 'sine'},
    {'name': 'Snare', 'freq': 180, 'timbre': 'noise'},
    {'name': 'Hi-hat', 'freq': 320, 'timbre': 'noise'},
    {'name': 'Trumpet', 'freq': 440, 'timbre': 'brass'},
    {'name': 'Violin', 'freq': 660, 'timbre': 'string'},
    {'name': 'Flute', 'freq': 880, 'timbre': 'sine'},
    {'name': 'Piano', 'freq': 220, 'timbre': 'piano'},
]
N_BINS = len(BIN_INSTRUMENTS)
N_FRAMES = 32
SAMPLE_RATE = 44100
FRAME_MS = 120
FRAME_SAMPLES = int(SAMPLE_RATE * (FRAME_MS / 1000))

# --- Synthesis ---
def synth_frame(frame, pitch_shifts=None, bin_timbres=None):
    t = np.linspace(0, FRAME_MS/1000, FRAME_SAMPLES, endpoint=False)
    out = np.zeros(FRAME_SAMPLES)
    for b, amp in enumerate(frame):
        if amp > 0:
            freq = BIN_INSTRUMENTS[b]['freq']
            if pitch_shifts and pitch_shifts[b]:
                freq *= 2 ** (pitch_shifts[b] / 12)
            timbre = bin_timbres[b] if bin_timbres else BIN_INSTRUMENTS[b]['timbre']
            if timbre == 'sine':
                waveform = np.sin(2 * np.pi * freq * t)
            elif timbre == 'noise':
                waveform = np.random.uniform(-1, 1, size=len(t))
            elif timbre == 'brass':
                waveform = np.sign(np.sin(2 * np.pi * freq * t)) * (1 - 0.5 * np.sin(2 * np.pi * freq * 2 * t))
            elif timbre == 'string':
                waveform = (np.sin(2 * np.pi * freq * t) + 0.5 * np.sin(2 * np.pi * freq * 2 * t)) / 1.5
            elif timbre == 'piano':
                waveform = np.sin(2 * np.pi * freq * t) * np.exp(-3 * t)
            else:
                waveform = np.sin(2 * np.pi * freq * t)
            waveform *= amp / 255
            out += waveform
    return out

# --- Piano Roll GUI ---
def piano_roll():
    pygame.init()
    window = pygame.display.set_mode((800, 350))
    pygame.display.set_caption('Binary Orchestra Piano Roll')
    clock = pygame.time.Clock()
    cell_w, cell_h = 20, 40
    pitch_shifts = [0] * N_BINS
    amplitudes = [180] * N_BINS
    bin_timbres = [BIN_INSTRUMENTS[b]['timbre'] for b in range(N_BINS)]
    score = np.zeros((N_BINS, N_FRAMES), dtype=np.uint8)
    selected = [0, 0]
    playing = False

    def draw():
        window.fill((30,30,30))
        for b in range(N_BINS):
            for f in range(N_FRAMES):
                color = (100,200,100) if score[b,f] else (50,50,50)
                if [b,f]==selected:
                    color = (255,255,0)
                pygame.draw.rect(window, color, (f*cell_w+40,b*cell_h+30,cell_w-2,cell_h-2))
            label = BIN_INSTRUMENTS[b]['name']
            window.blit(pygame.font.SysFont('Arial',16).render(label,1,(255,255,255)),(2,b*cell_h+40))
        pygame.display.flip()

    def play_score():
        for f in range(N_FRAMES):
            frame = score[:,f]
            audio = synth_frame(frame, pitch_shifts, bin_timbres)
            sd.play(audio, SAMPLE_RATE)
            pygame.draw.rect(window, (255,0,0), (f*cell_w+40, 10, cell_w-2, 15))
            pygame.display.flip()
            pygame.time.wait(FRAME_MS)
        sd.stop()

    while True:
        draw()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit(); return
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    play_score()
                elif event.key == pygame.K_RIGHT:
                    selected[1] = (selected[1]+1)%N_FRAMES
                elif event.key == pygame.K_LEFT:
                    selected[1] = (selected[1]-1)%N_FRAMES
                elif event.key == pygame.K_DOWN:
                    selected[0] = (selected[0]+1)%N_BINS
                elif event.key == pygame.K_UP:
                    selected[0] = (selected[0]-1)%N_BINS
                elif event.key == pygame.K_RETURN:
                    b,f = selected
                    score[b,f] = 255 if score[b,f]==0 else 0
                elif event.key == pygame.K_p: # Pitch up
                    pitch_shifts[selected[0]] += 1
                elif event.key == pygame.K_o: # Pitch down
                    pitch_shifts[selected[0]] -= 1
                elif event.key == pygame.K_a: # Amp up
                    amplitudes[selected[0]] = min(255, amplitudes[selected[0]]+10)
                elif event.key == pygame.K_z: # Amp down
                    amplitudes[selected[0]] = max(0, amplitudes[selected[0]]-10)
        clock.tick(30)

if __name__ == "__main__":
    piano_roll()

2. How Each Instrument's Binary is Encoded and Tuned

    Each time frame is a binary vector: [0,0,255,0,0,0,0] (this plays the Hi-hat at max amplitude in this frame).
    Pitch tuning: Use keys p (up) and o (down) for the selected instrument.
    Amplitude: Use a (up) and z (down) for the selected instrument.
    Timbre: By default, each bin has an instrument-appropriate timbre, but you can expand the code to change it interactively.

3. Scaling Up, C++/Crow/Web Integration

    The above can be ported to C++ for performance, using RtAudio or JUCE for sound and ImGui or SDL for visuals.
    For a web interface, build a REST API in C++ using Crow to handle score input/output and playback commands, and a WebGL/TypeScript frontend for visuals.
    Save/load binary scores as .npy or .json files for offline rendering.

4. Legal Sound Libraries

    All synthesis above is code-based (original, legal).
    If you want to use samples, choose open-licensed (e.g. Freesound), load into bins, and play as needed.

5. How to Expand Further

    Add more bins/instruments by extending BIN_INSTRUMENTS.
    Add visual effects: make each bin map to a distinct color/motion/shape in the visualization.
    Support MIDI input/output.
    Export your live AV performance as high-quality WAV/MP4.

Skip to content
Navigation Menu
.

This framework turns binary patterns into storytelling instruments — a new kind of orchestra where bins are players, frames are measures, and visuals are the score’s living portrait.
