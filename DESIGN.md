# Dantotsu Print — Design (QFD)

This document decomposes the goals of **Dantotsu Print** — a tool that lets a
developer put the *actual defective code* onto the **Defect** cell of a physical
A3 daily Dantotsu problem-solving board — into functions, approaches,
components, and the tradeoffs taken. Vocabulary is defined in
[CONTEXT.md](./CONTEXT.md).

**Scope.** Dantotsu Print owns only the *capture of a code defect onto the
Defect cell*. The broader outcome it serves — the daily board becoming the
team's living quality ritual, displacing Notion — is the **backdrop it enables
but does not own** (that depends on facilitation, huddle discipline, and
management buy-in, all outside this tool). The tool must not be judged by
whether the huddle happens.

**Deployment context.** ~400 engineers, 7 floors, **50+ boards by end of 2026**,
**per-board** printer unit. Yokoten is ~80% within the board's own team, ~15%
floor-level, ~5% company-wide.

Strength weights used in matrices: **9** strong, **3** medium, **1** weak, blank none.

---

## House of Quality

```tikz
\usetikzlibrary{arrows.meta, positioning, shapes.geometric, shapes.misc, calc, fit, backgrounds}

% Toggles
\newif\ifqfdshowroof          \qfdshowrooftrue
\newif\ifqfdshowbasement      \qfdshowbasementtrue
\newif\ifqfdshowcompetitive   \qfdshowcompetitivetrue
\newif\ifqfdshowlegend        \qfdshowlegendtrue
\newif\ifqfdshowimportance    \qfdshowimportancetrue
\newif\ifqfdshowcorrlegend    \qfdshowcorrlegendtrue
\newif\ifqfdshowevallegend    \qfdshowevallegendtrue
\newif\ifqfdshowtitle         \qfdshowtitletrue

% Dimensions
\def\qfdNW{5}
\def\qfdNH{5}
\def\qfdWhatW{4.0}
\def\qfdImpW{0.9}
\def\qfdCmpW{3}
\def\qfdHdrH{2.6}
\def\qfdBasementN{4}

% Titles & labels
\def\qfdWhatsTitle{Customer needs}
\def\qfdImpTitle{Imp.\ \%}
\def\qfdPerceptionTitle{Comparative evaluation}
\def\qfdPoorLabel{poor}
\def\qfdExcellentLabel{excellent}
\def\qfdAltOneLabel{Our product}
\def\qfdAltTwoLabel{Competitor A}
\def\qfdAltThreeLabel{Competitor B}
\def\qfdRelTitle{Relation}
\def\qfdCorrTitle{Correlation}
\def\qfdEvalTitle{Evaluation}

\def\qfdProjectTitle{}
\def\qfdConcept{}

% Styles
\tikzset{
  qfdthin/.style ={line width=0.35pt},
  qfdmed/.style  ={line width=0.7pt},
  qfdstrong/.style={circle, draw, fill=black,
                    minimum size=7pt, inner sep=0pt},
  qfdmod/.style  ={circle, draw,
                    minimum size=7pt, inner sep=0pt, line width=0.8pt},
  qfdweak/.style ={regular polygon, regular polygon sides=3, draw,
                    minimum size=8.5pt, inner sep=0pt, line width=0.7pt},
  qfdrel/.is choice,
  qfdrel/S/.style={qfdstrong},
  qfdrel/M/.style={qfdmod},
  qfdrel/W/.style={qfdweak},
  qfdalt1mk/.style={circle, draw, fill=black,
                    minimum size=6pt, inner sep=0pt, line width=1pt},
  qfdalt1ln/.style={line width=1.2pt},
  qfdalt2mk/.style={regular polygon, regular polygon sides=3, draw,
                    fill=black, minimum size=6pt, inner sep=0pt,
                    line width=0.7pt},
  qfdalt2ln/.style={line width=0.7pt, dashed},
  qfdalt3mk/.style={rectangle, draw, fill=black,
                    minimum size=5pt, inner sep=0pt, line width=0.7pt},
  qfdalt3ln/.style={line width=0.7pt, dotted},
}

\newcommand{\qfdDrawGrid}{%
  \foreach \c in {1,...,\qfdNHm} \draw[qfdthin] (\c, 0) -- (\c, -\qfdNW);
  \foreach \r in {1,...,\qfdNWm} \draw[qfdthin] (0, -\r) -- (\qfdNH, -\r);
  \foreach \r in {1,...,\qfdNWm}
    \draw[qfdthin] (\qfdLeftEdge, -\r) -- (0, -\r);
  \ifqfdshowroof
    \foreach \c in {1,...,\qfdNHm}
      \draw[qfdthin] (\c, 0) -- (\c, \qfdHdrH);
  \fi
  \ifqfdshowcompetitive
    \foreach \r in {1,...,\qfdNWm}
      \draw[qfdthin] (\qfdNH, -\r) -- (\qfdNH+\qfdCmpW, -\r);
  \fi
  \ifqfdshowbasement
    \foreach \r in {1,...,\qfdBasementN}
      \draw[qfdthin] (0, -\qfdNW-\r) -- (\qfdNH, -\qfdNW-\r);
    \foreach \c in {1,...,\qfdNHm}
      \draw[qfdthin] (\c, -\qfdNW) -- (\c, -\qfdNW-\qfdBasementN);
  \fi
}

\newcommand{\qfdDrawRoof}{%
  \ifqfdshowroof
    \foreach \k in {1,...,\qfdNHm} {%
      \pgfmathsetmacro{\rx}{(\k+\qfdNH)/2}
      \pgfmathsetmacro{\ry}{\qfdHdrH + (\qfdNH-\k)/2}
      \pgfmathsetmacro{\lx}{\k/2}
      \pgfmathsetmacro{\ly}{\qfdHdrH + \k/2}
      \draw[qfdthin] (\k, \qfdHdrH) -- (\rx, \ry);
      \draw[qfdthin] (\k, \qfdHdrH) -- (\lx, \ly);
    }%
    \draw[qfdmed] (0, \qfdHdrH)
       -- (\qfdNH/2, \qfdApexY) -- (\qfdNH, \qfdHdrH);
    \foreach \i in {1,...,\qfdNH}
      \foreach \k in {1,...,\qfdNH} {%
        \pgfmathtruncatemacro{\jj}{\i+\k}
        \ifnum\jj>\qfdNH\relax\else
          \pgfmathsetmacro{\xx}{\i + \k/2 - 0.5}
          \pgfmathsetmacro{\yy}{\qfdHdrH + \k/2}
          \coordinate (C-\i-\jj) at (\xx, \yy);
        \fi
      }%
  \fi
}

\newcommand{\qfdDrawScale}{%
  \ifqfdshowcompetitive
    \foreach \tk in {0,1,2,3,4,5} {%
      \pgfmathsetmacro{\tx}{\qfdNH + (\tk+0.5)*\qfdCmpW/6}
      \node[anchor=south, font=\scriptsize] at (\tx, 0.02) {\tk};
    }%
    \node[anchor=south, font=\scriptsize\bfseries, align=center]
         at ({\qfdNH + \qfdCmpW/2}, 0.7) {\qfdPerceptionTitle};
    \node[anchor=north, font=\scriptsize\itshape]
         at ({\qfdNH + 0.45}, -\qfdNW) {\qfdPoorLabel};
    \node[anchor=north, font=\scriptsize\itshape]
         at ({\qfdNH + \qfdCmpW - 0.45}, -\qfdNW) {\qfdExcellentLabel};
  \fi
}

\newcommand{\qfdDrawZoneTitles}{%
  \ifqfdshowimportance
    \node[rotate=90, anchor=west, font=\footnotesize\bfseries]
         at ({-\qfdImpW/2}, 0.12) {\qfdImpTitle};
  \fi
  \node[font=\scriptsize\bfseries, align=center, text width=\qfdWhatW cm]
       at ({\qfdLeftEdge + \qfdWhatW/2},
           {\ifqfdshowroof \qfdHdrH/2 \else 0.6 \fi}) {\qfdWhatsTitle};
}

\newcommand{\qfdDrawTitle}{%
  \ifqfdshowtitle
    \ifx\qfdProjectTitle\empty\else
      \pgfmathsetmacro{\qfdTitleX}{\qfdNH/2}
      \pgfmathsetmacro{\qfdTitleY}{\ifqfdshowroof \qfdApexY \else \qfdHdrH \fi + 0.9}
      \pgfmathsetmacro{\qfdSubW}{\qfdNH + 2}
      \node[anchor=south, font=\large\bfseries, align=center]
           at (\qfdTitleX, \qfdTitleY) {\qfdProjectTitle};
      \ifx\qfdConcept\empty\else
        \node[anchor=north, font=\footnotesize\itshape, align=center,
              text width=\qfdSubW cm]
             at (\qfdTitleX, {\qfdTitleY - 0.1}) {\qfdConcept};
      \fi
    \fi
  \fi
}

\newcommand{\qfdDrawFrames}{%
  \begin{scope}[qfdmed]
    \draw (\qfdLeftEdge, 0) rectangle (\qfdNH, -\qfdNW);
    \ifqfdshowimportance \draw (-\qfdImpW, 0) -- (-\qfdImpW, -\qfdNW); \fi
    \draw (0, 0) -- (0, -\qfdNW);
    \ifqfdshowroof
      \draw (0, 0) rectangle (\qfdNH, \qfdHdrH); \fi
    \ifqfdshowbasement
      \draw (0, -\qfdNW) rectangle (\qfdNH, -\qfdNW-\qfdBasementN); \fi
    \ifqfdshowcompetitive
      \draw (\qfdNH, 0) rectangle (\qfdNH+\qfdCmpW, -\qfdNW); \fi
  \end{scope}
}

\newcommand{\qfdDrawLegend}{%
  \ifqfdshowlegend
    \pgfmathsetmacro{\qfdLegX}{%
      \qfdNH + \ifqfdshowcompetitive \qfdCmpW + 0.7 \else 0.7 \fi}
    \pgfmathsetmacro{\qfdLegBottom}{%
      -2.05
      \ifqfdshowroof    \ifqfdshowcorrlegend - 2.55 \fi \fi
      \ifqfdshowcompetitive \ifqfdshowevallegend - 2.20 \fi \fi}
    \pgfmathsetmacro{\qfdLegY}{\qfdHdrH - 0.4}
    \begin{scope}[shift={(\qfdLegX, \qfdLegY)}]
      \draw[qfdmed, rounded corners=2pt]
        (-0.15, 0.4) rectangle (4.5, \qfdLegBottom);
      \node[anchor=west, font=\footnotesize\bfseries] at (0, 0.1)
        {\qfdRelTitle};
      \draw[qfdthin] (0, -0.15) -- (4.35, -0.15);
      \node[qfdstrong] at (0.22, -0.5)  {};
        \node[anchor=west] at (0.5, -0.5)  {Strong (9)};
      \node[qfdmod]    at (0.22, -0.95) {};
        \node[anchor=west] at (0.5, -0.95) {Medium (3)};
      \node[qfdweak]   at (0.22, -1.4)  {};
        \node[anchor=west] at (0.5, -1.4)  {Weak (1)};
      \ifqfdshowroof \ifqfdshowcorrlegend
        \node[anchor=west, font=\footnotesize\bfseries] at (0, -2.10)
          {\qfdCorrTitle};
        \draw[qfdthin] (0, -2.35) -- (4.35, -2.35);
        \node[anchor=west] at (0, -2.70) {{$+\!+$}\quad very positive};
        \node[anchor=west] at (0, -3.05) {{$+$\phantom{$+$}}\quad positive};
        \node[anchor=west] at (0, -3.40) {{$-$\phantom{$-$}}\quad negative};
        \node[anchor=west] at (0, -3.75) {{$-\!-$}\quad very negative};
      \fi \fi
      \ifqfdshowcompetitive \ifqfdshowevallegend
        \pgfmathsetmacro{\qfdEvalTop}{%
          -2.10 \ifqfdshowroof\ifqfdshowcorrlegend - 2.55 \fi\fi}
        \node[anchor=west, font=\footnotesize\bfseries]
          at (0, \qfdEvalTop) {\qfdEvalTitle};
        \pgfmathsetmacro{\qfdEvalSep}{\qfdEvalTop - 0.25}
        \draw[qfdthin] (0, \qfdEvalSep) -- (4.35, \qfdEvalSep);
        \pgfmathsetmacro{\qfdLegA}{\qfdEvalTop - 0.55}
        \draw[qfdalt1ln] (0.05, \qfdLegA) -- (0.45, \qfdLegA);
          \node[qfdalt1mk] at (0.25, \qfdLegA) {};
          \node[anchor=west, font=\bfseries] at (0.55, \qfdLegA)
            {\qfdAltOneLabel};
        \pgfmathsetmacro{\qfdLegB}{\qfdEvalTop - 0.95}
        \draw[qfdalt2ln] (0.05, \qfdLegB) -- (0.45, \qfdLegB);
          \node[qfdalt2mk] at (0.25, \qfdLegB) {};
          \node[anchor=west] at (0.55, \qfdLegB) {\qfdAltTwoLabel};
        \pgfmathsetmacro{\qfdLegC}{\qfdEvalTop - 1.35}
        \draw[qfdalt3ln] (0.05, \qfdLegC) -- (0.45, \qfdLegC);
          \node[qfdalt3mk] at (0.25, \qfdLegC) {};
          \node[anchor=west] at (0.55, \qfdLegC) {\qfdAltThreeLabel};
      \fi \fi
    \end{scope}
  \fi
}

\newenvironment{qfdhouse}{%
  \begin{tikzpicture}[x=1cm, y=1cm, font=\scriptsize,
                      line cap=round, line join=round]
  \ifqfdshowimportance
    \pgfmathsetmacro{\qfdLeftEdge}{-\qfdWhatW-\qfdImpW}
  \else
    \pgfmathsetmacro{\qfdLeftEdge}{-\qfdWhatW}
  \fi
  \pgfmathsetmacro{\qfdApexY}{\qfdHdrH + \qfdNH/2}
  \pgfmathtruncatemacro{\qfdNHm}{\qfdNH - 1}
  \pgfmathtruncatemacro{\qfdNWm}{\qfdNW - 1}
  \qfdDrawGrid
  \qfdDrawRoof
  \qfdDrawScale
  \qfdDrawZoneTitles
  \qfdDrawTitle
}{%
  \qfdDrawFrames
  \qfdDrawLegend
  \end{tikzpicture}%
}

% ---- Overrides for Dantotsu Print ----
\def\qfdNW{3}
\def\qfdNH{5}
\def\qfdWhatW{4.5}
\def\qfdHdrH{3.0}
\def\qfdImpTitle{Weight}
\def\qfdWhatsTitle{Goals (WHATs)}
\def\qfdProjectTitle{Dantotsu Print}
\def\qfdConcept{One \textbf{keystroke} turns selected code into a \textbf{peel-stick defect card} for the physical \textbf{Dantotsu board}.}
\qfdshowcompetitivefalse

\begin{document}
\begin{qfdhouse}
  % WHATs + weights
  \pgfmathsetmacro{\qfdWhatTextW}{\qfdWhatW - 0.2}
  \foreach \r/\t in {1/{Effortless, faithful capture},
                     2/{Deployable at 50+ boards},
                     3/{Traceable to source}}
    \node[anchor=west, font=\scriptsize, text width=\qfdWhatTextW cm, align=left]
      at ({\qfdLeftEdge + 0.1}, {-\r + 0.5}) {\t};
  \foreach \r/\imp in {1/10, 2/8, 3/6}
    \node[font=\scriptsize] at ({-\qfdImpW/2}, {-\r + 0.5}) {\imp};

  % HOWs
  \foreach \c/\t in {1/{Capture effort}, 2/{Fidelity \& legibility},
                     3/{Scope to cell}, 4/{Operating burden},
                     5/{Source back-link}}
    \node[rotate=90, anchor=west, font=\scriptsize] at ({\c - 0.5}, 0.15) {\t};

  % Relations
  \node[qfdrel/S] at ({1 - 0.5}, {-1 + 0.5}) {};
  \node[qfdrel/S] at ({2 - 0.5}, {-1 + 0.5}) {};
  \node[qfdrel/M] at ({3 - 0.5}, {-1 + 0.5}) {};
  \node[qfdrel/W] at ({4 - 0.5}, {-1 + 0.5}) {};
  \node[qfdrel/S] at ({4 - 0.5}, {-2 + 0.5}) {};
  \node[qfdrel/S] at ({5 - 0.5}, {-3 + 0.5}) {};

  % Roof correlations
  \node[font=\scriptsize] at (C-1-3) {$-$};
  \node[font=\scriptsize] at (C-1-4) {$+$};
  \node[font=\scriptsize] at (C-2-3) {$+$};

  % Basement row labels
  \node[anchor=west, font=\scriptsize] at ({\qfdLeftEdge + 0.1}, {-\qfdNW - 0.5}) {Target};
  \node[anchor=west, font=\scriptsize] at ({\qfdLeftEdge + 0.1}, {-\qfdNW - 1.5}) {Difficulty (1--5)};
  \node[anchor=west, font=\scriptsize] at ({\qfdLeftEdge + 0.1}, {-\qfdNW - 2.5}) {Abs.\ weight};
  \node[anchor=west, font=\scriptsize] at ({\qfdLeftEdge + 0.1}, {-\qfdNW - 3.5}) {Rel.\ weight \%};

  % Basement values
  \foreach \c/\tgt/\diff/\abs/\rel in
    {1/{$\leq$10s}/4/90/26,
     2/{48$\times$19}/3/90/26,
     3/{preview}/2/30/9,
     4/{unbox}/4/82/24,
     5/{@sha}/2/54/16} {
    \node[font=\scriptsize] at ({\c - 0.5}, {-\qfdNW - 0.5}) {\tgt};
    \node[font=\scriptsize] at ({\c - 0.5}, {-\qfdNW - 1.5}) {\diff};
    \node[font=\scriptsize] at ({\c - 0.5}, {-\qfdNW - 2.5}) {\abs};
    \node[font=\scriptsize\bfseries] at ({\c - 0.5}, {-\qfdNW - 3.5}) {\rel};
  }
\end{qfdhouse}
\end{document}
```

---

## 1. Goals — the WHATs

| ID  | Goal                                                                                                                          | Weight | Source |
|-----|-------------------------------------------------------------------------------------------------------------------------------|:------:|--------|
| G1  | Getting the **real defective code onto the Defect cell** is effortless and faithful — so filling the sheet is no longer the chore that sends people back to Notion | 10 | walk-with-me 2026-07-01 |
| G2  | Deployable to **50+ boards** at low TCO, *without harming adoption* (weight is operational burden, not the modest cash cost)   |   8    | walk-with-me 2026-07-01 |
| G3  | A defect on the board stays **traceable to its exact source** (commit + line) — earns its keep in the ~20% cross-team yokoten cases | 6 | walk-with-me 2026-07-01 |

## 2. Functions — the HOWs

Direction: ↑ higher is better, ↓ lower is better, → hit a fixed target.

| ID  | Function                                                                                                  | Dir | Target (now) |
|-----|-----------------------------------------------------------------------------------------------------------|:---:|--------------|
| F1  | **Minimise capture effort** — gestures & seconds from "I see the defect" to "card in hand"                 |  ↓  | ≤ 1 gesture after selecting code (one keystroke); card in hand ≤ 10 s |
| F2  | **Preserve code fidelity & legibility** — verbatim monospaced code inside the 90×85 mm cell | ↑ | exact characters; readable up-close (~0.4 m), card *present* at ~1 m; fits the mono grid budget (~48 cols × ~19 rows at 8 pt); overflow *flagged*, never silently shrunk |
| F3  | **Scope the defect to the cell** — developer previews & trims the selection to the essential offending lines |  →  | live preview of exact print output; adjustable line range; flags (not shrinks) content beyond the legible budget |
| F4  | **Minimise per-board operating burden** — consumables, refills, jams, provisioning handled by non-experts, no babysitting | ↓ | consumable swap ≤ monthly & self-serve; per-unit setup ≤ "unbox and pair"; no per-card manual step beyond peel/stick |
| F5  | **Embed a durable back-link to source** — every card carries a *computer-native* reference to the exact code | → | printed text `owner/repo · path · Lstart–end · @short-sha` (commit-pinned), read straight into the editor; **optional** QR to the full web permalink for at-the-board phone glances only — never the transfer path |

_F1–F3 serve G1, F4 serves G2, F5 serves G3 (mapped in §3)._

_**F5 later upgrade (deferred from v1):** an SSO-backed short-link redirector (`go/dp/AB12` → full permalink) for one-click open on the computer. Cheap to maintain once set up behind existing SSO (auth already solved; a lookup table). Trigger to build: cross-team yokoten (the ~20%) shows the typed `path:line@sha` reference is too slow in practice._

## 3. Cascade — Goals → Functions → How → Components

_Pipeline stages resolved so far; printer/transport (F4) and F3's preview mechanism still open._

- **G1** Effortless & faithful defect capture  _W:10_
  - **F1** Minimise capture effort  _↓ ≤1 gesture, ≤10 s_
    - **How (chosen):** one **CLI engine** invoked by a **per-editor keystroke** that pipes the selection and passes `--file`/`--lines` (Vim map, JetBrains External Tool, VS Code/Cursor keybinding→task). **Xcode:** clipboard-only fallback, no line numbers. _Chosen over a per-editor extension — see T1._
  - **F2** Fidelity & legibility  _↑_
    - **How (chosen):** CLI renders selection → image via its own tokenizer (tree-sitter / `silicon`-class), sized to the 90×85 mm cell. Monospace makes fit a deterministic **character budget** (~48 cols × ~19 rows at 8 pt after margins + footer); the CLI flags selections that exceed it. Font picked for legibility at the printer's DPI (label ≈ 300 dpi). The defect itself is **marked** on the card — one or more character-precise spans (box / bold / highlight / underline), sourced from the editor selection(s); see [SPEC.md](./SPEC.md).
  - **F3** Scope to the cell  _→_
    - **How (chosen):** F3a — CLI renders the real card **at exact cell size (90×85 mm), cell boundary drawn**, and previews it: **inline** in graphics-capable terminals (iTerm2 / Kitty / **Ghostty** / WezTerm / VS Code-Cursor sixel), or as a **text-grid** (code clamped to the ~48-col budget — instant, no graphics needed), or the PNG opened in the OS viewer. Overflow spills past the border → F2's "flag, don't shrink" made visual. Confirm-to-print; `-y` skips. WYSIWYG because the preview *is* the print artifact.
- **G3** Traceable to source  _W:6_
  - **F5** Durable back-link  _→_
    - **How (chosen):** CLI derives the introducing commit SHA via `git blame` on the selected lines; prints `repo·path·Lx–y·@sha` + optional QR.
- **G2** Deployable at scale  _W:8_
  - **F4** Minimise per-board operating burden  _↓_
    - **How (chosen — peel-stick label):** per-board **monochrome direct-thermal label printer**, wide **networked Brother QL-1110NWB (102 mm)**, ≥300 dpi, **removable** adhesive (peel-on-resolve won't tear the A3). Bonded to the cell → no fall/misfile (G1 trust), zero manual step (F1), no ink (F4). CLI-driven via `brother_ql`. _Rejected: magnet (fall/misfile), receipt+glue (per-card step) — see T3 / §9._
    - **How (chosen — transport):** **T-B central print service** (thin router) — CLI renders client-side, then POSTs *card image + board-id* to the service, which forwards the raster to that board's **networked Brother QL-1110NWB** by IP. Service owns the board→printer registry, retries, and print/label status. _Over direct (T-A) and corporate CUPS (T-C) — see T4._

## 4. House — Goals × Functions

Cells: link strength (9/3/1/blank). Importance row = Σ(weight × strength).

|          | F1 | F2 | F3 | F4 | F5 |
|----------|:--:|:--:|:--:|:--:|:--:|
| G1 (10)  | 9  | 9  | 3  | 1  |    |
| G2 (8)   |    |    |    | 9  |    |
| G3 (6)   |    |    |    |    | 9  |
| **Σ**    | 90 | 90 | 30 | 82 | 54 |

**Top engineering priorities:** **F1 capture-effort (90) ≈ F2 fidelity (90)** are the heart of G1 — pour engineering there. **F4 operating-burden (82)** is a close third because it is the *sole* driver of the heavy G2. **F5 (54)** is mid. **F3 scope/preview (30)** is the least critical function *and* it conflicts with F1 (see roof) — so build it light; don't gold-plate the preview.

## 5. Roof — Function × Function

`◎` strong reinforce · `○` mild reinforce · `×` mild conflict · `⊗` strong conflict.

|        | F1 | F2 | F3 | F4 | F5 |
|--------|:--:|:--:|:--:|:--:|:--:|
| **F1** | —  |    | ×  | ○  |    |
| **F2** |    | —  | ○  |    |    |
| **F3** |    |    | —  |    |    |
| **F4** |    |    |    | —  |    |
| **F5** |    |    |    |    | —  |

**Conflicts that shape the design:**
- **F1 × F3 (×)** — the confirm-preview adds a gesture against F1's ≤1-gesture target. Mitigated per-invocation by `-y`. This is the one live tension (see §8).
- **F2 × F3 (○)** — scoping the snippet to the cell *helps* fit and legibility; they reinforce.
- **F1 × F4 (○)** — the central service's `--board` routing removes per-print printer-hunting, mildly aiding capture effort.

## 6. Components & Function → Component map

The second house: Functions (HOWs) become the WHATs, realised by concrete components.

| ID  | Component                                                                 | ADR      |
|-----|---------------------------------------------------------------------------|----------|
| C1  | `dp` CLI engine — **Rust, `clap`** (capture stdin+`--file`/`--lines`, render, preview, emit) | ADR-0001, ADR-0003 |
| C2  | Per-editor keystroke shims (Vim / JetBrains / VS Code·Cursor; Xcode clipboard) | ADR-0001 |
| C3  | Tokenizing renderer — **`syntect`/`silicon`-class → mono PNG at cell size**, `viuer` inline preview | ADR-0003 |
| C4  | Central print service — **Python / `brother_ql`** (board→printer registry, queue, status, router) | ADR-0002, ADR-0004 |
| C5  | Networked label printer — Brother QL-1110NWB, per board                    | ADR-0002 |
| C6  | Board registry / config (`--board`, per-repo/dev default)                 | ADR-0002 |

Function → Component strength (9/3/1/blank):

|    | C1 | C2 | C3 | C4 | C5 | C6 |
|----|:--:|:--:|:--:|:--:|:--:|:--:|
| F1 | 9  | 9  |    |    |    | 3  |
| F2 | 3  |    | 9  |    |    |    |
| F3 | 9  |    | 3  |    |    |    |
| F4 |    |    |    | 9  | 9  | 3  |
| F5 | 9  |    |    | 1  |    |    |

_F5's deferred SSO short-link redirector would later attach to **C4**._

### House 2 — Functions × Components

Functions (House 1's HOWs) are now the WHATs, down the left with their carried-down
weights; components are the HOWs across the top. Roof = component↔component
correlation; basement = per-component target, build difficulty, and weight.

<!-- Self-contained: this ```tikz fence compiles independently, so it repeats
     House 1's template preamble verbatim; only the overrides + body below differ. -->

```tikz
\usetikzlibrary{arrows.meta, positioning, shapes.geometric, shapes.misc, calc, fit, backgrounds}

% Toggles
\newif\ifqfdshowroof          \qfdshowrooftrue
\newif\ifqfdshowbasement      \qfdshowbasementtrue
\newif\ifqfdshowcompetitive   \qfdshowcompetitivetrue
\newif\ifqfdshowlegend        \qfdshowlegendtrue
\newif\ifqfdshowimportance    \qfdshowimportancetrue
\newif\ifqfdshowcorrlegend    \qfdshowcorrlegendtrue
\newif\ifqfdshowevallegend    \qfdshowevallegendtrue
\newif\ifqfdshowtitle         \qfdshowtitletrue

% Dimensions
\def\qfdNW{5}
\def\qfdNH{5}
\def\qfdWhatW{4.0}
\def\qfdImpW{0.9}
\def\qfdCmpW{3}
\def\qfdHdrH{2.6}
\def\qfdBasementN{4}

% Titles & labels
\def\qfdWhatsTitle{Customer needs}
\def\qfdImpTitle{Imp.\ \%}
\def\qfdPerceptionTitle{Comparative evaluation}
\def\qfdPoorLabel{poor}
\def\qfdExcellentLabel{excellent}
\def\qfdAltOneLabel{Our product}
\def\qfdAltTwoLabel{Competitor A}
\def\qfdAltThreeLabel{Competitor B}
\def\qfdRelTitle{Relation}
\def\qfdCorrTitle{Correlation}
\def\qfdEvalTitle{Evaluation}

\def\qfdProjectTitle{}
\def\qfdConcept{}

% Styles
\tikzset{
  qfdthin/.style ={line width=0.35pt},
  qfdmed/.style  ={line width=0.7pt},
  qfdstrong/.style={circle, draw, fill=black,
                    minimum size=7pt, inner sep=0pt},
  qfdmod/.style  ={circle, draw,
                    minimum size=7pt, inner sep=0pt, line width=0.8pt},
  qfdweak/.style ={regular polygon, regular polygon sides=3, draw,
                    minimum size=8.5pt, inner sep=0pt, line width=0.7pt},
  qfdrel/.is choice,
  qfdrel/S/.style={qfdstrong},
  qfdrel/M/.style={qfdmod},
  qfdrel/W/.style={qfdweak},
  qfdalt1mk/.style={circle, draw, fill=black,
                    minimum size=6pt, inner sep=0pt, line width=1pt},
  qfdalt1ln/.style={line width=1.2pt},
  qfdalt2mk/.style={regular polygon, regular polygon sides=3, draw,
                    fill=black, minimum size=6pt, inner sep=0pt,
                    line width=0.7pt},
  qfdalt2ln/.style={line width=0.7pt, dashed},
  qfdalt3mk/.style={rectangle, draw, fill=black,
                    minimum size=5pt, inner sep=0pt, line width=0.7pt},
  qfdalt3ln/.style={line width=0.7pt, dotted},
}

\newcommand{\qfdDrawGrid}{%
  \foreach \c in {1,...,\qfdNHm} \draw[qfdthin] (\c, 0) -- (\c, -\qfdNW);
  \foreach \r in {1,...,\qfdNWm} \draw[qfdthin] (0, -\r) -- (\qfdNH, -\r);
  \foreach \r in {1,...,\qfdNWm}
    \draw[qfdthin] (\qfdLeftEdge, -\r) -- (0, -\r);
  \ifqfdshowroof
    \foreach \c in {1,...,\qfdNHm}
      \draw[qfdthin] (\c, 0) -- (\c, \qfdHdrH);
  \fi
  \ifqfdshowcompetitive
    \foreach \r in {1,...,\qfdNWm}
      \draw[qfdthin] (\qfdNH, -\r) -- (\qfdNH+\qfdCmpW, -\r);
  \fi
  \ifqfdshowbasement
    \foreach \r in {1,...,\qfdBasementN}
      \draw[qfdthin] (0, -\qfdNW-\r) -- (\qfdNH, -\qfdNW-\r);
    \foreach \c in {1,...,\qfdNHm}
      \draw[qfdthin] (\c, -\qfdNW) -- (\c, -\qfdNW-\qfdBasementN);
  \fi
}

\newcommand{\qfdDrawRoof}{%
  \ifqfdshowroof
    \foreach \k in {1,...,\qfdNHm} {%
      \pgfmathsetmacro{\rx}{(\k+\qfdNH)/2}
      \pgfmathsetmacro{\ry}{\qfdHdrH + (\qfdNH-\k)/2}
      \pgfmathsetmacro{\lx}{\k/2}
      \pgfmathsetmacro{\ly}{\qfdHdrH + \k/2}
      \draw[qfdthin] (\k, \qfdHdrH) -- (\rx, \ry);
      \draw[qfdthin] (\k, \qfdHdrH) -- (\lx, \ly);
    }%
    \draw[qfdmed] (0, \qfdHdrH)
       -- (\qfdNH/2, \qfdApexY) -- (\qfdNH, \qfdHdrH);
    \foreach \i in {1,...,\qfdNH}
      \foreach \k in {1,...,\qfdNH} {%
        \pgfmathtruncatemacro{\jj}{\i+\k}
        \ifnum\jj>\qfdNH\relax\else
          \pgfmathsetmacro{\xx}{\i + \k/2 - 0.5}
          \pgfmathsetmacro{\yy}{\qfdHdrH + \k/2}
          \coordinate (C-\i-\jj) at (\xx, \yy);
        \fi
      }%
  \fi
}

\newcommand{\qfdDrawScale}{%
  \ifqfdshowcompetitive
    \foreach \tk in {0,1,2,3,4,5} {%
      \pgfmathsetmacro{\tx}{\qfdNH + (\tk+0.5)*\qfdCmpW/6}
      \node[anchor=south, font=\scriptsize] at (\tx, 0.02) {\tk};
    }%
    \node[anchor=south, font=\scriptsize\bfseries, align=center]
         at ({\qfdNH + \qfdCmpW/2}, 0.7) {\qfdPerceptionTitle};
    \node[anchor=north, font=\scriptsize\itshape]
         at ({\qfdNH + 0.45}, -\qfdNW) {\qfdPoorLabel};
    \node[anchor=north, font=\scriptsize\itshape]
         at ({\qfdNH + \qfdCmpW - 0.45}, -\qfdNW) {\qfdExcellentLabel};
  \fi
}

\newcommand{\qfdDrawZoneTitles}{%
  \ifqfdshowimportance
    \node[rotate=90, anchor=west, font=\footnotesize\bfseries]
         at ({-\qfdImpW/2}, 0.12) {\qfdImpTitle};
  \fi
  \node[font=\scriptsize\bfseries, align=center, text width=\qfdWhatW cm]
       at ({\qfdLeftEdge + \qfdWhatW/2},
           {\ifqfdshowroof \qfdHdrH/2 \else 0.6 \fi}) {\qfdWhatsTitle};
}

\newcommand{\qfdDrawTitle}{%
  \ifqfdshowtitle
    \ifx\qfdProjectTitle\empty\else
      \pgfmathsetmacro{\qfdTitleX}{\qfdNH/2}
      \pgfmathsetmacro{\qfdTitleY}{\ifqfdshowroof \qfdApexY \else \qfdHdrH \fi + 0.9}
      \pgfmathsetmacro{\qfdSubW}{\qfdNH + 2}
      \node[anchor=south, font=\large\bfseries, align=center]
           at (\qfdTitleX, \qfdTitleY) {\qfdProjectTitle};
      \ifx\qfdConcept\empty\else
        \node[anchor=north, font=\footnotesize\itshape, align=center,
              text width=\qfdSubW cm]
             at (\qfdTitleX, {\qfdTitleY - 0.1}) {\qfdConcept};
      \fi
    \fi
  \fi
}

\newcommand{\qfdDrawFrames}{%
  \begin{scope}[qfdmed]
    \draw (\qfdLeftEdge, 0) rectangle (\qfdNH, -\qfdNW);
    \ifqfdshowimportance \draw (-\qfdImpW, 0) -- (-\qfdImpW, -\qfdNW); \fi
    \draw (0, 0) -- (0, -\qfdNW);
    \ifqfdshowroof
      \draw (0, 0) rectangle (\qfdNH, \qfdHdrH); \fi
    \ifqfdshowbasement
      \draw (0, -\qfdNW) rectangle (\qfdNH, -\qfdNW-\qfdBasementN); \fi
    \ifqfdshowcompetitive
      \draw (\qfdNH, 0) rectangle (\qfdNH+\qfdCmpW, -\qfdNW); \fi
  \end{scope}
}

\newcommand{\qfdDrawLegend}{%
  \ifqfdshowlegend
    \pgfmathsetmacro{\qfdLegX}{%
      \qfdNH + \ifqfdshowcompetitive \qfdCmpW + 0.7 \else 0.7 \fi}
    \pgfmathsetmacro{\qfdLegBottom}{%
      -2.05
      \ifqfdshowroof    \ifqfdshowcorrlegend - 2.55 \fi \fi
      \ifqfdshowcompetitive \ifqfdshowevallegend - 2.20 \fi \fi}
    \pgfmathsetmacro{\qfdLegY}{\qfdHdrH - 0.4}
    \begin{scope}[shift={(\qfdLegX, \qfdLegY)}]
      \draw[qfdmed, rounded corners=2pt]
        (-0.15, 0.4) rectangle (4.5, \qfdLegBottom);
      \node[anchor=west, font=\footnotesize\bfseries] at (0, 0.1)
        {\qfdRelTitle};
      \draw[qfdthin] (0, -0.15) -- (4.35, -0.15);
      \node[qfdstrong] at (0.22, -0.5)  {};
        \node[anchor=west] at (0.5, -0.5)  {Strong (9)};
      \node[qfdmod]    at (0.22, -0.95) {};
        \node[anchor=west] at (0.5, -0.95) {Medium (3)};
      \node[qfdweak]   at (0.22, -1.4)  {};
        \node[anchor=west] at (0.5, -1.4)  {Weak (1)};
      \ifqfdshowroof \ifqfdshowcorrlegend
        \node[anchor=west, font=\footnotesize\bfseries] at (0, -2.10)
          {\qfdCorrTitle};
        \draw[qfdthin] (0, -2.35) -- (4.35, -2.35);
        \node[anchor=west] at (0, -2.70) {{$+\!+$}\quad very positive};
        \node[anchor=west] at (0, -3.05) {{$+$\phantom{$+$}}\quad positive};
        \node[anchor=west] at (0, -3.40) {{$-$\phantom{$-$}}\quad negative};
        \node[anchor=west] at (0, -3.75) {{$-\!-$}\quad very negative};
      \fi \fi
      \ifqfdshowcompetitive \ifqfdshowevallegend
        \pgfmathsetmacro{\qfdEvalTop}{%
          -2.10 \ifqfdshowroof\ifqfdshowcorrlegend - 2.55 \fi\fi}
        \node[anchor=west, font=\footnotesize\bfseries]
          at (0, \qfdEvalTop) {\qfdEvalTitle};
        \pgfmathsetmacro{\qfdEvalSep}{\qfdEvalTop - 0.25}
        \draw[qfdthin] (0, \qfdEvalSep) -- (4.35, \qfdEvalSep);
        \pgfmathsetmacro{\qfdLegA}{\qfdEvalTop - 0.55}
        \draw[qfdalt1ln] (0.05, \qfdLegA) -- (0.45, \qfdLegA);
          \node[qfdalt1mk] at (0.25, \qfdLegA) {};
          \node[anchor=west, font=\bfseries] at (0.55, \qfdLegA)
            {\qfdAltOneLabel};
        \pgfmathsetmacro{\qfdLegB}{\qfdEvalTop - 0.95}
        \draw[qfdalt2ln] (0.05, \qfdLegB) -- (0.45, \qfdLegB);
          \node[qfdalt2mk] at (0.25, \qfdLegB) {};
          \node[anchor=west] at (0.55, \qfdLegB) {\qfdAltTwoLabel};
        \pgfmathsetmacro{\qfdLegC}{\qfdEvalTop - 1.35}
        \draw[qfdalt3ln] (0.05, \qfdLegC) -- (0.45, \qfdLegC);
          \node[qfdalt3mk] at (0.25, \qfdLegC) {};
          \node[anchor=west] at (0.55, \qfdLegC) {\qfdAltThreeLabel};
      \fi \fi
    \end{scope}
  \fi
}

\newenvironment{qfdhouse}{%
  \begin{tikzpicture}[x=1cm, y=1cm, font=\scriptsize,
                      line cap=round, line join=round]
  \ifqfdshowimportance
    \pgfmathsetmacro{\qfdLeftEdge}{-\qfdWhatW-\qfdImpW}
  \else
    \pgfmathsetmacro{\qfdLeftEdge}{-\qfdWhatW}
  \fi
  \pgfmathsetmacro{\qfdApexY}{\qfdHdrH + \qfdNH/2}
  \pgfmathtruncatemacro{\qfdNHm}{\qfdNH - 1}
  \pgfmathtruncatemacro{\qfdNWm}{\qfdNW - 1}
  \qfdDrawGrid
  \qfdDrawRoof
  \qfdDrawScale
  \qfdDrawZoneTitles
  \qfdDrawTitle
}{%
  \qfdDrawFrames
  \qfdDrawLegend
  \end{tikzpicture}%
}

% ---- Overrides for House 2 (Functions × Components) ----
\def\qfdNW{5}
\def\qfdNH{6}
\def\qfdWhatW{4.7}
\def\qfdHdrH{3.2}
\def\qfdImpTitle{Weight}
\def\qfdWhatsTitle{Functions (WHATs)}
\def\qfdProjectTitle{Dantotsu Print — House 2}
\def\qfdConcept{Functions (the HOWs of House 1) realised by concrete \textbf{components}.}
\qfdshowcompetitivefalse

\begin{document}
\begin{qfdhouse}
  % WHATs (Functions) + carried-down weights (House 1 relative weights)
  \pgfmathsetmacro{\qfdWhatTextW}{\qfdWhatW - 0.2}
  \foreach \r/\t in {1/{F1 Minimise capture effort},
                     2/{F2 Fidelity \& legibility},
                     3/{F3 Scope to the cell},
                     4/{F4 Minimise operating burden},
                     5/{F5 Durable back-link}}
    \node[anchor=west, font=\scriptsize, text width=\qfdWhatTextW cm, align=left]
      at ({\qfdLeftEdge + 0.1}, {-\r + 0.5}) {\t};
  \foreach \r/\imp in {1/26, 2/26, 3/9, 4/24, 5/16}
    \node[font=\scriptsize] at ({-\qfdImpW/2}, {-\r + 0.5}) {\imp};

  % HOWs (Components)
  \foreach \c/\t in {1/{C1 CLI engine}, 2/{C2 Keystroke shims},
                     3/{C3 Renderer}, 4/{C4 Print service},
                     5/{C5 Label printer}, 6/{C6 Board registry}}
    \node[rotate=90, anchor=west, font=\scriptsize] at ({\c - 0.5}, 0.15) {\t};

  % Relations (Function x Component, from the section 6 matrix)
  \node[qfdrel/S] at ({1 - 0.5}, {-1 + 0.5}) {}; % F1-C1
  \node[qfdrel/S] at ({2 - 0.5}, {-1 + 0.5}) {}; % F1-C2
  \node[qfdrel/M] at ({6 - 0.5}, {-1 + 0.5}) {}; % F1-C6
  \node[qfdrel/M] at ({1 - 0.5}, {-2 + 0.5}) {}; % F2-C1
  \node[qfdrel/S] at ({3 - 0.5}, {-2 + 0.5}) {}; % F2-C3
  \node[qfdrel/S] at ({1 - 0.5}, {-3 + 0.5}) {}; % F3-C1
  \node[qfdrel/M] at ({3 - 0.5}, {-3 + 0.5}) {}; % F3-C3
  \node[qfdrel/S] at ({4 - 0.5}, {-4 + 0.5}) {}; % F4-C4
  \node[qfdrel/S] at ({5 - 0.5}, {-4 + 0.5}) {}; % F4-C5
  \node[qfdrel/M] at ({6 - 0.5}, {-4 + 0.5}) {}; % F4-C6
  \node[qfdrel/S] at ({1 - 0.5}, {-5 + 0.5}) {}; % F5-C1
  \node[qfdrel/W] at ({4 - 0.5}, {-5 + 0.5}) {}; % F5-C4

  % Roof correlations (Component x Component)
  \node[font=\scriptsize] at (C-1-2) {$+\!+$}; % C1-C2 shims wrap the CLI
  \node[font=\scriptsize] at (C-1-3) {$+\!+$}; % C1-C3 renderer in the CLI
  \node[font=\scriptsize] at (C-1-4) {$+$};    % C1-C4 CLI POSTs to service
  \node[font=\scriptsize] at (C-1-6) {$+$};    % C1-C6 CLI reads --board
  \node[font=\scriptsize] at (C-3-4) {$+$};    % C3-C4 service transports raster
  \node[font=\scriptsize] at (C-3-5) {$-$};    % C3-C5 mono printer caps renderer
  \node[font=\scriptsize] at (C-4-5) {$+\!+$}; % C4-C5 service drives printer
  \node[font=\scriptsize] at (C-4-6) {$+\!+$}; % C4-C6 registry lives in service
  \node[font=\scriptsize] at (C-5-6) {$+$};    % C5-C6 registry maps board->printer

  % Basement row labels
  \node[anchor=west, font=\scriptsize] at ({\qfdLeftEdge + 0.1}, {-\qfdNW - 0.5}) {Target};
  \node[anchor=west, font=\scriptsize] at ({\qfdLeftEdge + 0.1}, {-\qfdNW - 1.5}) {Difficulty (1--5)};
  \node[anchor=west, font=\scriptsize] at ({\qfdLeftEdge + 0.1}, {-\qfdNW - 2.5}) {Abs.\ weight};
  \node[anchor=west, font=\scriptsize] at ({\qfdLeftEdge + 0.1}, {-\qfdNW - 3.5}) {Rel.\ weight \%};

  % Basement values
  \foreach \c/\tgt/\diff/\abs/\rel in
    {1/{$\leq$1 s}/4/537/33,
     2/{4 eds}/2/234/14,
     3/{300 dpi}/4/261/16,
     4/{route}/3/232/14,
     5/{102 mm}/1/216/13,
     6/{board}/2/150/9} {
    \node[font=\scriptsize] at ({\c - 0.5}, {-\qfdNW - 0.5}) {\tgt};
    \node[font=\scriptsize] at ({\c - 0.5}, {-\qfdNW - 1.5}) {\diff};
    \node[font=\scriptsize] at ({\c - 0.5}, {-\qfdNW - 2.5}) {\abs};
    \node[font=\scriptsize\bfseries] at ({\c - 0.5}, {-\qfdNW - 3.5}) {\rel};
  }
\end{qfdhouse}
\end{document}
```

**Reading it.** **C1 (33%)** dominates — it realises four of five functions; it is
the hub, and with **C3 (16%)** carries the build risk (both Difficulty 4). C2/C4/C5
cluster ~13–14%, C6 trails at 9%. Effort should track weight.

#### House 2 roof — Component × Component

`◎` strong reinforce · `○` mild reinforce · `×` mild conflict.

|        | C1 | C2 | C3 | C4 | C5 | C6 |
|--------|:--:|:--:|:--:|:--:|:--:|:--:|
| **C1** | —  | ◎  | ◎  | ○  |    | ○  |
| **C2** |    | —  |    |    |    |    |
| **C3** |    |    | —  | ○  | ×  |    |
| **C4** |    |    |    | —  | ◎  | ◎  |
| **C5** |    |    |    |    | —  | ○  |
| **C6** |    |    |    |    |    | —  |

- **C1×C2 (◎), C1×C3 (◎)** — the shims are thin wrappers that invoke the CLI, and the renderer is a library inside it; building the CLI *is* building the seat these sit in.
- **C4×C5 (◎), C4×C6 (◎)** — the print service drives the printer via `brother_ql` and owns the board→printer registry; one coherent server.
- **C3×C5 (×) — the one conflict.** A richer renderer is capped by the monochrome ≥300 dpi / 102 mm label: hue and fine detail C3 *could* produce don't survive the print. This is **T2** (§8) made structural — spend renderer effort on legible monochrome, not colour.
- **Mild reinforces (○):** C1×C4 (CLI POSTs the card), C1×C6 (CLI reads `--board` to route), C3×C4 (service transports C3's raster — they must agree on its spec), C5×C6 (registry maps board→printer IP).

_Watched, not a roof conflict: **C1 (Rust) ↔ C4 (Python)** cooperate to print, but the two runtimes couple across an HTTP contract that must not drift — see **T6** (§8)._

#### House 2 basement — component performance budget

_Abs./Rel. weight are **computed** — §6 matrix × carried-down function weights
(F1 26, F2 26, F3 9, F4 24, F5 16). **Target** and **Difficulty** are a first
assessment introduced with this house; revise as ADRs/spikes land._

| Component | Target (now) | Difficulty | Abs. | Rel. % |
|-----------|--------------|:----------:|:----:|:------:|
| C1 CLI engine     | single static binary; cold run ≤ ~1 s to emit                        | 4 | 537 | **33** |
| C3 Renderer       | verbatim mono PNG at 90×85 mm, ~48c×19r @ 8 pt, crisp @300 dpi, span marks | 4 | 261 | **16** |
| C2 Keystroke shims| one map per editor (Vim/JetBrains/VS Code·Cursor); Xcode clipboard   | 2 | 234 | **14** |
| C4 Print service  | board→printer route + retries + status; service up                   | 3 | 232 | **14** |
| C5 Label printer  | Brother QL-1110NWB, 102 mm, ≥300 dpi, removable, networked           | 1 | 216 | **13** |
| C6 Board registry | `--board` + per-repo/dev default; board→printer map                  | 2 | 150 | **9**  |

## 7. Critical performance budget

Ranked from §4 importance and §5 conflicts. `If we miss it` is mandatory — a target without a fallback is a wish.

| Rank | Function | Target | Watched on | If we miss it |
|------|----------|--------|------------|---------------|
| 1 | **F1** capture effort | ≤ 1 gesture, ≤ 10 s | dogfood timing on pilot boards | Notion-relapse risk → default `-y`, tighten keybindings, revisit shims |
| 2 | **F2** fidelity/legibility | verbatim; ~48c × 19r @ 8 pt, crisp at 300 dpi | print tests read at the board | drop to 7 pt / widen media / enforce F3 trim |
| 3 | **F4** operating burden | unbox+pair; consumable swap ≤ monthly & self-serve; service up | refill cadence + service uptime + jam/label-out incidents across pilot boards | Pi-agent fallback, pre-stock labels, service HA |
| 4 | **F5** back-link | resolves to the introducing commit | sample cards resolve to correct SHA | fix blame-vs-HEAD; build the SSO redirector |
| 5 | **F3** scope/preview | preview accurate; overflow flagged not shrunk | overflow-flag correctness on oversized snippets | low stakes → fall back to re-select-and-refire |

## 8. Tradeoffs — Got / Paid / ADR

| ID  | Tradeoff | Got | Paid | ADR |
|-----|----------|-----|------|-----|
| T1  | One **CLI engine + per-editor keystroke shims**, over a per-editor extension | editor-agnostic across the polyglot fleet (VS Code, Cursor, JetBrains, Vim, Xcode); single engine to maintain (serves F4) | weaker in-editor preview for F3; Xcode degraded to clipboard-only; syntax colors from CLI tokenizer, not the editor's live theme | ADR-0001 |
| T2  | **Monochrome print**, over color on paper | no ink/toner refills across 50 boards (F4); high-contrast distance legibility | no syntax *hue* on the card — kept only in the preview + digital permalink | — |
| T3  | **Peel-stick label** (wide networked QL-1110NWB, removable), over magnet and over receipt+glue | bonded → no fall/misfile (G1 trust); zero manual step (F1); no ink (F4); CLI-driven via `brother_ql` | ~€9k/50 + recurring label rolls; wide model needed for width | — |
| T4  | **T-B central service + networked printers**, over direct (T-A) / corporate CUPS (T-C) | central board→printer registry, retries, print/label status, `--board` routing, home for the F5 redirector; no per-board Pi to patch | a small service to run + maintain; single point of failure if down | ADR-0002 |
| T5  | **Rust plain CLI**, over TS/Node and Gleam, no TUI framework | single-binary distribution to the polyglot fleet (F4); best-fit code→PNG ecosystem; no TUI machinery on the fire-and-exit path | Rust-comfortable maintainers required; interactive trim deferred to a scoped ratatui mode | ADR-0003 |
| T6  | **C4 in Python (FastAPI + `brother_ql`)**, over Node/Hono, Deno/Oak, Gleam | native, battle-tested QL raster driver; one runtime; zero glue | Python in a TS-heavy shop; a second language beside the Rust CLI (clean HTTP boundary) | ADR-0004 |

### Tensions being watched (unresolved by design)

- **Central print service is a single point of failure.** If it's down, printing stalls. Instead: keep it dead-simple / HA, and let the CLI cache the registry to fall back to direct-print. **Trigger to revisit:** first outage that blocks a daily huddle, or flaky uptime.
- **F1 × F3 — preview vs. speed.** The confirm-preview adds a gesture against F1's ≤1-gesture target. Instead: `-y` skips preview per-invocation. **Trigger to revisit:** if devs habitually `-y` everything, drop preview-by-default.

## 9. Inconsistencies spotted and fixed

- **Goal overreach — "the board becomes the ritual."** First-draft G1 claimed the tool revives the whole daily practice. Rescoped: the tool owns only *capture onto the Defect cell*; the ritual is backdrop it enables but does not own (see Scope).
- **"Defect = the crash."** The switch-missing-a-case example blended the code defect with its consequence (the crash). Resolved: the **Defect** is the code; the crash is only the consequence that made it visible, and is not printed.
- **F5 assumed a QR/phone.** Traceability was first specced as a QR — but the audience is developers at computers, and a QR resolves to a phone. Resolved: primary back-link is computer-native `path·lines·@sha` text; QR demoted to an optional at-the-board glance.
- **F2 "legible at ~1 m" over-specified.** Dense code is read up-close (~0.4 m); at 1 m only the card's *presence* registers. Target corrected.
- **Printer class closed too early, then re-derived correctly.** Originally ruled out thermal receipt on a false premise ("needs glue"). Reopened: Brother QL is itself direct-thermal (the thermal/label split was false; fading is moot at Daily), and the magnetic board suggested magnet attach. Magnet was then eliminated on an **integrity failure mode** — a fallen card re-stuck on the wrong row silently corrupts the defect↔cause/owner link (a G1 trust regression). Landed back on **peel-stick label**, now justified by *bonding/integrity*, not the original hand-wave.

---

## How to keep this honest

- New ADR lands → add its components to §6 and re-score affected rows.
- A spike / measurement returns numbers → update §7 `Target` / `Watched on`.
- WHATs (§1) change rarely; HOWs (§2) change per release; matrices (§§4–6) are recomputed when either side changes.
- If a section becomes empty after edits, delete it — empty sections lie.
