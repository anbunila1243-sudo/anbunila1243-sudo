## Hi there 👋

<!--
**anbunila1243-sudo/anbunila1243-sudo** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
import os, random, zipfile, xml.dom.minidom as md
from html import escape

BG0, BG1, CARD, LINE, ACC, ACC2, SOFT, TEXT, MUTED = "#060B18", "#0B1530", "#0F1D3F", "#1E3A7A", "#2F6BFF", "#5B9BFF", "#A9C7FF", "#EAF2FF", "#8DA2C8"
SERIF = "Georgia,'Times New Roman',serif"
SANS = "'Segoe UI','Helvetica Neue',Arial,sans-serif"
MONO = "'SF Mono',Consolas,'Courier New',monospace"
USER = "anbunila1243-sudo"
OUT = os.path.dirname(os.path.abspath(__file__))
A = os.path.join(OUT, "assets")
os.makedirs(A, exist_ok=True)

STYLE = """.fade{animation:fade 1.1s ease-out backwards}@keyframes fade{from{opacity:0}to{opacity:1}}
.card{animation:glowb 5s ease-in-out infinite}@keyframes glowb{0%,100%{stroke:LINE;stroke-opacity:.85}50%{stroke:ACC2;stroke-opacity:1}}
.pulse{animation:pulse 3s ease-in-out infinite}@keyframes pulse{0%,100%{opacity:.45}50%{opacity:1}}
.float{animation:float 4s ease-in-out infinite}@keyframes float{0%,100%{transform:translateY(0)}50%{transform:translateY(-9px)}}
.spin{animation:spin 28s linear infinite;transform-box:fill-box;transform-origin:center}@keyframes spin{to{transform:rotate(360deg)}}
.tw{animation:tw 3s ease-in-out infinite}@keyframes tw{0%,100%{opacity:.08}50%{opacity:.95}}
.shine{transform:translateX(-400px);animation:shine 8s ease-in-out infinite}@keyframes shine{0%{transform:translateX(-300px)}55%,100%{transform:translateX(WIDTHpx)}}
.blink{animation:blink 1s steps(1) infinite}@keyframes blink{50%{opacity:0}}
.line{animation:lineIn .6s ease-out backwards}@keyframes lineIn{from{opacity:0;transform:translateX(-14px)}to{opacity:1;transform:translateX(0)}}
.ecg{stroke-dasharray:90 260;animation:ecg 2.6s linear infinite}@keyframes ecg{from{stroke-dashoffset:350}to{stroke-dashoffset:0}}"""


def style(w):
    return STYLE.replace("WIDTH", str(w + 300)).replace("LINE", LINE).replace("ACC2", ACC2).replace("SOFT", SOFT)


def E(s):
    return escape(s, quote=False)


def svg(w, h, body, defs=""):
    return f'''<svg xmlns="http://www.w3.org/2000/svg" width="{w}" height="{h}" viewBox="0 0 {w} {h}">
<defs><style>{style(w)}</style>
<linearGradient id="bg" x1="0" y1="0" x2="1" y2="1"><stop offset="0" stop-color="{BG0}"/><stop offset="1" stop-color="{BG1}"/></linearGradient>
<pattern id="dots" width="24" height="24" patternUnits="userSpaceOnUse"><circle cx="2" cy="2" r="1" fill="{LINE}" fill-opacity=".55"/></pattern>
<linearGradient id="sg" x1="0" y1="0" x2="1" y2="0"><stop offset="0" stop-color="{SOFT}" stop-opacity="0"/><stop offset=".5" stop-color="{SOFT}" stop-opacity=".13"/><stop offset="1" stop-color="{SOFT}" stop-opacity="0"/></linearGradient>
<clipPath id="rc"><rect width="{w}" height="{h}" rx="22"/></clipPath>
<filter id="glow" x="-30%" y="-40%" width="160%" height="180%"><feGaussianBlur stdDeviation="4" result="b"><animate attributeName="stdDeviation" values="2;6;2" dur="3.5s" repeatCount="indefinite"/></feGaussianBlur><feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge></filter>
{defs}</defs>
<rect width="{w}" height="{h}" rx="22" fill="url(#bg)"/>
<rect width="{w}" height="{h}" rx="22" fill="url(#dots)"/>
<rect x=".5" y=".5" width="{w-1}" height="{h-1}" rx="22" fill="none" stroke="{LINE}"/>
<rect x="2" y="2" width="{w-4}" height="{h-4}" rx="21" fill="none" stroke="{ACC2}" stroke-width="1.5" class="pulse"/>
<g clip-path="url(#rc)"><polygon class="shine" points="0,{h} 70,0 170,0 100,{h}" fill="url(#sg)"/></g>
<g class="fade">{body}</g>
</svg>'''


def T(x, y, s, size=16, fill=TEXT, font=SANS, w="normal", anchor="start", extra=""):
    return f'<text x="{x}" y="{y}" font-family="{font}" font-size="{size}" font-weight="{w}" fill="{fill}" text-anchor="{anchor}" {extra}>{E(s)}</text>'


def card(x, y, w, h, d=0):
    return f'<rect class="card" x="{x}" y="{y}" width="{w}" height="{h}" rx="16" fill="{CARD}" stroke="{LINE}" style="animation-delay:{d}s"/>'


def chip(x, y, s):
    w = round(len(s) * 7.6 + 26)
    return (f'<rect x="{x}" y="{y}" width="{w}" height="30" rx="15" fill="{CARD}" stroke="{LINE}"/>'
            + T(x + w / 2, y + 20, s, 13, SOFT, SANS, "bold", "middle")), w


def flow(items, x0, y0, maxx, gap=8, rowh=40):
    out, x, y = "", x0, y0
    for s in items:
        w = round(len(s) * 7.6 + 26)
        if x + w > maxx:
            x, y = x0, y + rowh
        c, _ = chip(x, y, s)
        out += c
        x += w + gap
    return out


def ecg(y, x1=60, x2=1140):
    d = f"M{x1} {y} H540 l12 -22 l14 44 l14 -22 H{x2}"
    return (f'<path d="{d}" fill="none" stroke="{ACC}" stroke-opacity=".25" stroke-width="2"/>'
            f'<path class="ecg" d="{d}" fill="none" stroke="{ACC2}" stroke-width="2.5" stroke-linecap="round" filter="url(#glow)"/>')


files = {}

# ---------- hero ----------
random.seed(7)
tw = "".join(f'<circle class="tw" cx="{random.randint(30,1170)}" cy="{random.randint(30,530)}" r="{random.choice([1.5,2,2.5])}" fill="{SOFT}" style="animation-delay:{round(random.uniform(0,3),2)}s"/>' for _ in range(26))
chips, x = "", 60
for s in ["React.js", "JavaScript", "Python", "MySQL"]:
    c, w = chip(x, 330, s)
    chips += c
    x += w + 10
pills = ""
for i, (s, px, py) in enumerate([("React.js", 700, 60), ("MySQL", 1010, 60), ("PHP", 760, 500)]):
    c, _ = chip(px, py, s)
    pills += f'<g class="float" style="animation-delay:{i*0.8}s">{c}</g>'
hero = (tw
        + T(60, 95, "FULL STACK DEVELOPER ASPIRANT", 15, ACC2, MONO, "bold", extra='letter-spacing="3"')
        + T(60, 190, "D. Anbu", 96, TEXT, SERIF, "bold", extra='filter="url(#glow)"')
        + '<g clip-path="url(#tc)">'
        + T(60, 255, "Responsive interfaces,", 26, SOFT, SERIF, extra='font-style="italic"')
        + T(60, 290, "backed by solid databases.", 26, SOFT, SERIF, extra='font-style="italic"')
        + '</g>'
        + f'<rect class="blink" x="474" y="268" width="3" height="28" fill="{ACC2}" filter="url(#glow)"/>'
        + chips
        + T(60, 410, "B.Tech Artificial Intelligence and Data Science", 15, TEXT)
        + T(60, 440, "Rrase College of Engineering, Chennai | Expected 2027", 15, MUTED)
        + f'<circle class="pulse" cx="900" cy="285" r="215" fill="{ACC}" fill-opacity=".14" filter="url(#glow)"/>'
        + f'<circle class="spin" cx="900" cy="285" r="208" fill="none" stroke="{ACC2}" stroke-width="2" stroke-dasharray="6 10"/>'
        + f'<circle cx="900" cy="285" r="196" fill="none" stroke="{ACC2}" stroke-width="3" filter="url(#glow)"/>'
        + f'<circle cx="900" cy="285" r="190" fill="{CARD}"/>'
        + T(900, 338, "DA", 150, ACC2, SERIF, "bold", "middle", 'filter="url(#glow)"')
        + pills)
hdefs = '<clipPath id="tc"><rect x="60" y="225" width="540" height="90"><animate attributeName="width" from="0" to="540" dur="1.8s" begin="0s" fill="freeze"/></rect></clipPath>'
files["hero.svg"] = svg(1200, 560, hero, hdefs)

# ---------- typing ----------
phrases = ["Full Stack Developer Aspirant", "React.js | JavaScript | Python", "HTML5 | CSS3 | PHP | MySQL",
           "Full Stack Development Intern at Novitech", "Open to frontend internships"]
px0 = 340
defs, body = "", ""
for i, p in enumerate(phrases):
    W = len(p) * 12
    s = i * 0.2
    kt = [(s, 0), (s + 0.07, W), (s + 0.15, W), (s + 0.19, 0), (1, 0)]
    if s == 0:
        kt = kt[1:]
        kt = [(0, 0)] + [(0.07, W), (0.15, W), (0.19, 0), (1, 0)]
    kts = ";".join(str(round(t, 3)) for t, _ in kt)
    if i == 0:
        kts = "0;0.07;0.15;0.19;1"
    vals = ";".join(str(v) for _, v in kt)
    base = W if i == 0 else 0
    defs += f'<clipPath id="p{i}"><rect x="{px0}" y="30" width="{base}" height="40"><animate attributeName="width" values="{vals}" keyTimes="{kts}" dur="18s" repeatCount="indefinite"/></rect></clipPath>'
    body += f'<g clip-path="url(#p{i})">' + T(px0, 58, p, 20, TEXT, MONO) + '</g>'
cx_vals = []
for i, p in enumerate(phrases):
    pass
# cursor follows the text end: one animate with all phrases
vals, kts = [], []
for i, p in enumerate(phrases):
    W = len(p) * 12
    s = i * 0.2
    pts = [(s, 0), (s + 0.07, W), (s + 0.15, W), (s + 0.19, 0)]
    if i == 0:
        pts = pts[1:]
        pts = [(0, 0)] + pts[0:]
        pts = [(0, 0), (0.07, W), (0.15, W), (0.19, 0)]
    for t, v in pts:
        if kts and t <= kts[-1]:
            t = kts[-1] + 0.0005
        kts.append(round(t, 4))
        vals.append(px0 + v)
kts.append(1)
vals.append(px0)
cur = (f'<rect x="{px0}" y="36" width="3" height="28" fill="{ACC2}" filter="url(#glow)" class="blink"><animate attributeName="x" values="{";".join(map(str,vals))}" keyTimes="{";".join(map(str,kts))}" dur="18s" repeatCount="indefinite"/></rect>')
dots = "".join(f'<circle cx="{36+i*22}" cy="48" r="6" fill="{c}"/>' for i, c in enumerate([ACC, ACC2, SOFT]))
typing = dots + T(110, 58, "anbu@portfolio ~ $", 20, ACC2, MONO, "bold") + body + cur
files["typing.svg"] = svg(1200, 96, typing, defs)

# ---------- titles ----------
titles = [("about", "01 - ABOUT", "Who I Am"), ("work", "02 - SELECTED WORK", "Projects I Have Built"),
          ("experience", "03 - EXPERIENCE", "Hands-On Internship"), ("toolkit", "04 - TOOLKIT", "Tools I Build With"),
          ("certifications", "05 - CERTIFICATIONS", "Credentials Earned"), ("commit-log", "06 - COMMIT LOG", "Project Log"),
          ("connect", "07 - CONNECT", "Let's Build Together")]
for n, lab, head in titles:
    x1 = 50 + int(len(head) * 23) + 30
    b = (T(50, 42, lab, 14, ACC2, MONO, "bold", extra='letter-spacing="3"')
         + T(50, 92, head, 38, TEXT, SERIF, "bold", extra='filter="url(#glow)"')
         + f'<line x1="{x1}" y1="70" x2="1130" y2="70" stroke="{ACC2}" stroke-width="2" filter="url(#glow)"/>'
         + f'<circle class="pulse" cx="1148" cy="70" r="6" fill="{ACC2}" filter="url(#glow)"/>')
    files[f"title-{n}.svg"] = svg(1200, 120, b)

# ---------- about ----------
bio = ["B.Tech student in AI and Data Science who builds",
       "responsive, user-friendly web apps with HTML5, CSS3,",
       "JavaScript and React.js, backed by a Python foundation.",
       "Full Stack Development Intern at Novitech, working with",
       "PHP and MySQL on CRUD features and database integration.",
       "Seeking a frontend internship or entry-level role."]
b = "".join(T(50, 90 + i * 40, s, 18, TEXT if i < 5 else SOFT) for i, s in enumerate(bio))
stats = [("3", "Projects built"), ("1", "Internship"), ("2027", "Graduation (exp.)"), ("89%", "12th grade, 2024")]
for i, (v, l) in enumerate(stats):
    x, y = 870 + (i % 2) * 148, 40 + (i // 2) * 170
    b += card(x, y, 138, 150, i * 0.6)
    b += T(x + 69, y + 80, v, 40, ACC2, SERIF, "bold", "middle", 'filter="url(#glow)"')
    b += T(x + 69, y + 115, l, 12, MUTED, SANS, "normal", "middle")
files["about-intro.svg"] = svg(1200, 400, b)

# ---------- projects ----------
projects = [
    ("food-delivery", "Food Delivery Web Application", ["React.js", "HTML", "CSS", "JavaScript"],
     "A responsive food delivery interface with menu browsing and cart.",
     [("Dynamic menu", "Menu browsing"), ("Shopping cart", "Cart functionality"), ("Mobile-friendly", "Responsive interface")]),
    ("skybook", "SkyBook - Notes Management Application", ["HTML", "CSS", "JavaScript"],
     "A responsive notes-taking web app with a clean, intuitive interface.",
     [("CRUD", "Create, edit, organize notes"), ("Clean UI", "Intuitive interface"), ("Responsive", "Notes-taking web app")]),
    ("student-management", "Student Management System", ["PHP", "MySQL"],
     "A full stack app managing attendance, marks, fees and records.",
     [("Full stack", "Attendance, marks, fees"), ("CRUD + MySQL", "Database integration"), ("Records", "Student records management")]),
]
for i, (slug, name, stack, desc, mets) in enumerate(projects):
    b = T(50, 58, f"PROJECT 0{i+1}", 14, ACC2, MONO, "bold", extra='letter-spacing="3"')
    b += T(50, 108, name, 32, TEXT, SERIF, "bold", extra='filter="url(#glow)"')
    b += flow(stack, 50, 130, 1150)
    b += T(50, 205, desc, 17, SOFT)
    for j, (big, small) in enumerate(mets):
        x = 50 + j * 380
        b += card(x, 240, 340, 90, j * 0.7)
        b += T(x + 24, 282, big, 26, ACC2, SERIF, "bold")
        b += T(x + 24, 310, small, 13, MUTED)
    files[f"project-{i+1}.svg"] = svg(1200, 360, b)

# ---------- experience ----------
b = card(50, 20, 1100, 260, 0) + T(80, 62, "ROLE 01", 14, ACC2, MONO, "bold", extra='letter-spacing="3"')
b += T(80, 105, "Full Stack Development Intern", 30, TEXT, SERIF, "bold", extra='filter="url(#glow)"')
c, cw = chip(0, 0, "Novitech")
b += f'<g transform="translate({1120-cw},72)">{c}</g>'
for i, s in enumerate(["Developed and maintained web apps with HTML, CSS, JavaScript, PHP and MySQL.",
                       "Performed CRUD operations and database integration, building backend skills.",
                       "Collaborated with team members to debug and resolve technical issues."]):
    b += f'<g class="line" style="animation-delay:{round(0.45*(i+1),2)}s"><circle cx="90" cy="{155+i*35}" r="4" fill="{ACC2}"/>' + T(108, 161 + i * 35, s, 17, SOFT) + '</g>'
b += ecg(262, 80, 1120)
files["experience.svg"] = svg(1200, 300, b)

# ---------- toolkit ----------
groups = [("LANGUAGES", ["JavaScript", "HTML5", "CSS3", "Python"], 50, 30, 350),
          ("FRONTEND", ["React.js", "Bootstrap", "Responsive Design"], 425, 30, 350),
          ("TOOLS & PLATFORMS", ["Git", "GitHub", "VS Code"], 800, 30, 350),
          ("BACKEND & DATABASE", ["PHP", "MySQL", "CRUD Operations"], 50, 250, 540),
          ("CORE COMPETENCIES", ["Problem Solving", "Debugging", "Team Collaboration", "Time Management"], 610, 250, 540)]
b = ""
for i, (t, items, x, y, w) in enumerate(groups):
    b += card(x, y, w, 190, i * 0.5) + T(x + 20, y + 38, t, 13, ACC2, MONO, "bold", extra='letter-spacing="2"')
    b += flow(items, x + 20, y + 62, x + w - 20)
files["tech-constellation.svg"] = svg(1200, 470, b)

# ---------- certifications ----------
certs = [("01", ["Full Stack Development", "Internship"], "Novitech"), ("02", ["Mental Health", "First Aider"], "Mpower")]
b = ""
for i, (n, lines, org) in enumerate(certs):
    x = 50 + i * 560
    b += card(x, 40, 540, 250, i * 0.6) + T(x + 30, 130, n, 54, ACC2, SERIF, "bold", extra='filter="url(#glow)"')
    for j, l in enumerate(lines):
        b += T(x + 120, 110 + j * 34, l, 26, TEXT, SERIF, "bold")
    b += T(x + 120, 200, "Issued by " + org, 15, MUTED)
    b += f'<line x1="{x+120}" y1="222" x2="{x+500}" y2="222" stroke="{ACC2}" stroke-width="2" class="pulse"/>'
files["certifications.svg"] = svg(1200, 330, b)

# ---------- commit log ----------
b = card(50, 20, 1100, 400, 0)
b += "".join(f'<circle cx="{80+i*22}" cy="50" r="6" fill="{c}"/>' for i, c in enumerate([ACC, ACC2, SOFT]))
b += T(150, 55, f"anbu@portfolio: ~/projects", 13, MUTED, MONO)
rows = [("$ ls ~/projects", None)]
rows += [(p[1], (", ".join(p[2]), d)) for p, d in zip(projects, ["Responsive UI with menu browsing and cart", "Notes app with CRUD create, edit, organize", "Full stack records app: CRUD on MySQL"])]
ys = [105, 150, 195, 240]
delay = 0
for (n, extra), y in zip(rows, ys):
    delay += 0.45
    if extra is None:
        g = T(80, y, n, 15, ACC2, MONO, "bold")
    else:
        g = T(80, y, n, 15, TEXT, MONO, "bold") + T(440, y, extra[0], 15, ACC2, MONO) + T(660, y, extra[1], 15, MUTED, MONO)
    b += f'<g class="line" style="animation-delay:{round(delay,2)}s">{g}</g>'
b += f'<g class="line" style="animation-delay:2.3s">' + T(80, 285, "$ cat experience.txt", 15, ACC2, MONO, "bold") + '</g>'
b += f'<g class="line" style="animation-delay:2.75s">' + T(80, 325, "Novitech", 15, TEXT, MONO, "bold") + T(440, 325, "PHP, MySQL, JavaScript", 15, ACC2, MONO) + T(660, 325, "Full Stack Development Intern", 15, MUTED, MONO) + '</g>'
b += f'<g class="line" style="animation-delay:3.2s">' + T(80, 365, f"$ github.com/{USER}", 15, ACC2, MONO, "bold") + f'<rect class="blink" x="{80+9*(len(USER)+13)+6}" y="350" width="9" height="18" fill="{ACC2}" filter="url(#glow)"/></g>'
b += ecg(402, 80, 1120)
files["commit-log.svg"] = svg(1200, 440, b)

# ---------- footer ----------
b = T(600, 85, "D. Anbu", 64, TEXT, SERIF, "bold", "middle", 'filter="url(#glow)"')
b += T(600, 122, "Clean, intuitive interfaces. Solid backends.", 18, SOFT, SERIF, "normal", "middle", 'font-style="italic"')
b += T(600, 160, f"anbunila1243@email.com  |  linkedin.com/in/anbunila18  |  github.com/{USER}", 14, MUTED, MONO, "normal", "middle")
b += ecg(196)
files["footer-signature.svg"] = svg(1200, 220, b)

# ---------- write + validate ----------
for n, s in files.items():
    md.parseString(s.encode("utf-8"))
    with open(os.path.join(A, n), "w", encoding="utf-8") as f:
        f.write(s)

# ---------- README ----------
def img(f, alt):
    return f'<img src="https://raw.githubusercontent.com/{USER}/{USER}/main/assets/{f}?v=1" alt="{alt}" width="100%"/>'


bd = lambda l, v, u: f'<a href="{u}"><img src="https://img.shields.io/badge/{l}-{v}-2F6BFF?style=for-the-badge&labelColor=060B18" alt="{l}"/></a>'
readme = [img("hero.svg", "D. Anbu, Full Stack Developer Aspirant"), img("typing.svg", "Animated terminal banner with roles and stack"),
          '<p align="center">' + bd("LinkedIn", "anbunila18", "https://linkedin.com/in/anbunila18") + " "
          + bd("GitHub", "anbunila1243--sudo", f"https://github.com/{USER}") + " "
          + bd("Email", "anbunila1243@email.com", "mailto:anbunila1243@email.com") + "</p>"]
sec = [("about", "About", ["about-intro.svg"]), ("work", "Selected work", ["project-1.svg", "project-2.svg", "project-3.svg"]),
       ("experience", "Experience", ["experience.svg"]), ("toolkit", "Toolkit", ["tech-constellation.svg"]),
       ("certifications", "Certifications", ["certifications.svg"]), ("commit-log", "Commit log", ["commit-log.svg"])]
for n, t, fl in sec:
    readme.append(img(f"title-{n}.svg", f"Section title: {t}"))
    for f in fl:
        readme.append(img(f, f"{t} panel"))
readme.append(img("title-connect.svg", "Section title: Connect"))
readme.append(readme[2])
readme.append(img("footer-signature.svg", "D. Anbu signature footer"))
txt = "\n\n".join(readme) + "\n"
open(os.path.join(OUT, "README.md"), "w", encoding="utf-8").write(txt)
import re
for f in re.findall(r"assets/([\w\-\.]+\.svg)", txt):
    assert os.path.exists(os.path.join(A, f)), f

with zipfile.ZipFile(os.path.join(OUT, "profile-readme.zip"), "w", zipfile.ZIP_DEFLATED) as z:
    z.write(os.path.join(OUT, "README.md"), "README.md")
    for n in sorted(os.listdir(A)):
        z.write(os.path.join(A, n), "assets/" + n)
print("OK", len(files), "svgs")

