# ats-cv-builder
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>ATS CV Builder & Analyser</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- html2pdf for direct client-side PDF export -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
  <style>
    @media print {
      body * { visibility: hidden; }
      #cv-preview, #cv-preview * { visibility: visible; }
      #cv-preview { position: absolute; left: 0; top: 0; width: 100%; }
    }
  </style>
</head>
<body class="bg-slate-100 min-h-screen text-slate-800">
  <!-- Top Navigation -->
  <header class="bg-white border-b border-slate-200 px-6 py-4 flex justify-between items-center sticky top-0 z-20">
    <div class="font-bold text-xl text-slate-900 tracking-tight">CV Builder & ATS Analyser</div>
    <div class="flex gap-3">
      <button onclick="analyzeATS()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-md font-medium text-sm transition">
        Run ATS Check
      </button>
      <button onclick="downloadPDF()" class="bg-slate-900 hover:bg-slate-800 text-white px-4 py-2 rounded-md font-medium text-sm transition">
        Download PDF
      </button>
    </div>
  </header>

  <main class="max-w-7xl mx-auto p-6 grid grid-cols-1 lg:grid-cols-12 gap-6">
    <!-- Left Column: Editor & ATS Input -->
    <div class="lg:col-span-6 space-y-6">
      <!-- ATS Job Description Matcher -->
      <div class="bg-white p-5 rounded-lg border border-slate-200 shadow-sm">
        <h2 class="text-base font-semibold mb-2">1. Target Job Description (ATS Scanner)</h2>
        <textarea id="job-desc" rows="4" placeholder="Paste the target job description here to check keyword match..." 
          class="w-full p-2.5 text-sm border border-slate-300 rounded focus:ring-2 focus:ring-indigo-500 focus:outline-none"></textarea>
        
        <!-- Score Bar -->
        <div id="ats-results" class="mt-4 hidden space-y-2">
          <div class="flex justify-between text-sm font-medium">
            <span>ATS Match Score</span>
            <span id="score-text">0%</span>
          </div>
          <div class="w-full bg-slate-200 rounded-full h-2.5">
            <div id="score-bar" class="bg-indigo-600 h-2.5 rounded-full" style="width: 0%"></div>
          </div>
          <p id="missing-keywords" class="text-xs text-slate-600 pt-1"></p>
        </div>
      </div>

      <!-- CV Form Inputs -->
      <div class="bg-white p-5 rounded-lg border border-slate-200 shadow-sm space-y-4">
        <h2 class="text-base font-semibold">2. Personal Details</h2>
        <div class="grid grid-cols-2 gap-3">
          <input type="text" id="input-name" placeholder="Full Name" value="Jane Doe" oninput="updateCV()" class="p-2 border rounded text-sm w-full" />
          <input type="text" id="input-title" placeholder="Job Title" value="Senior Machine Learning Engineer" oninput="updateCV()" class="p-2 border rounded text-sm w-full" />
          <input type="email" id="input-email" placeholder="Email" value="jane.doe@example.com" oninput="updateCV()" class="p-2 border rounded text-sm w-full" />
          <input type="text" id="input-phone" placeholder="Phone" value="+44 20 7946 0912" oninput="updateCV()" class="p-2 border rounded text-sm w-full" />
          <input type="text" id="input-location" placeholder="Location" value="London, UK" oninput="updateCV()" class="col-span-2 p-2 border rounded text-sm w-full" />
        </div>

        <h2 class="text-base font-semibold pt-2">3. Professional Summary</h2>
        <textarea id="input-summary" rows="3" oninput="updateCV()" class="w-full p-2 border rounded text-sm">Results-driven Machine Learning Engineer with 6+ years of experience designing scalable deep learning pipelines, Bayesian probabilistic models, and computer vision systems.</textarea>

        <h2 class="text-base font-semibold pt-2">4. Core Skills (Comma-separated)</h2>
        <input type="text" id="input-skills" value="Python, PyTorch, Scikit-Learn, Bayesian Inference, Docker, AWS, MLOps, Data Pipelines" oninput="updateCV()" class="w-full p-2 border rounded text-sm" />

        <h2 class="text-base font-semibold pt-2">5. Work Experience</h2>
        <textarea id="input-experience" rows="5" oninput="updateCV()" class="w-full p-2 border rounded text-sm font-mono text-xs">Lead Machine Learning Engineer | DeepTech Labs | 2022 - Present
- Architected predictive models processing 2M+ daily events with 99.4% uptime.
- Reduced computer vision model training latency by 35% using distributed PyTorch.

Machine Learning Developer | DataCorp | 2019 - 2022
- Implemented Bayesian probabilistic frameworks for real-time anomaly detection.
- Built automated continuous integration pipelines for model deployment using Docker.</textarea>

        <h2 class="text-base font-semibold pt-2">6. Education</h2>
        <textarea id="input-education" rows="2" oninput="updateCV()" class="w-full p-2 border rounded text-sm">M.Sc. in Computer Science | Imperial College London | 2018 - 2019
B.Sc. in Mathematics | University of Bristol | 2015 - 2018</textarea>
      </div>
    </div>

    <!-- Right Column: Live ATS-Compliant CV Preview -->
    <div class="lg:col-span-6">
      <div class="sticky top-24">
        <div id="cv-preview" class="bg-white p-8 border border-slate-300 shadow-md min-h-[750px] font-sans text-slate-900 leading-relaxed">
          <!-- Header -->
          <div class="border-b border-slate-300 pb-4 mb-4 text-center">
            <h1 id="cv-name" class="text-2xl font-bold tracking-tight">Jane Doe</h1>
            <p id="cv-title" class="text-sm font-semibold text-slate-600 mt-0.5">Senior Machine Learning Engineer</p>
            <div id="cv-contacts" class="text-xs text-slate-500 mt-2 space-x-2">
              <span id="cv-email">jane.doe@example.com</span> •
              <span id="cv-phone">+44 20 7946 0912</span> •
              <span id="cv-location">London, UK</span>
            </div>
          </div>

          <!-- Summary -->
          <section class="mb-4">
            <h2 class="text-xs font-bold uppercase tracking-wider text-slate-900 border-b border-slate-200 pb-1 mb-1.5">Professional Summary</h2>
            <p id="cv-summary" class="text-xs text-slate-700"></p>
          </section>

          <!-- Skills -->
          <section class="mb-4">
            <h2 class="text-xs font-bold uppercase tracking-wider text-slate-900 border-b border-slate-200 pb-1 mb-1.5">Technical & Core Skills</h2>
            <p id="cv-skills" class="text-xs text-slate-700"></p>
          </section>

          <!-- Experience -->
          <section class="mb-4">
            <h2 class="text-xs font-bold uppercase tracking-wider text-slate-900 border-b border-slate-200 pb-1 mb-1.5">Experience</h2>
            <div id="cv-experience" class="text-xs text-slate-700 whitespace-pre-line space-y-2"></div>
          </section>

          <!-- Education -->
          <section class="mb-4">
            <h2 class="text-xs font-bold uppercase tracking-wider text-slate-900 border-b border-slate-200 pb-1 mb-1.5">Education</h2>
            <div id="cv-education" class="text-xs text-slate-700 whitespace-pre-line"></div>
          </section>
        </div>
      </div>
    </div>
  </main>

  <script>
    // Live CV Update
    function updateCV() {
      document.getElementById('cv-name').innerText = document.getElementById('input-name').value;
      document.getElementById('cv-title').innerText = document.getElementById('input-title').value;
      document.getElementById('cv-email').innerText = document.getElementById('input-email').value;
      document.getElementById('cv-phone').innerText = document.getElementById('input-phone').value;
      document.getElementById('cv-location').innerText = document.getElementById('input-location').value;
      document.getElementById('cv-summary').innerText = document.getElementById('input-summary').value;
      document.getElementById('cv-skills').innerText = document.getElementById('input-skills').value;
      document.getElementById('cv-experience').innerText = document.getElementById('input-experience').value;
      document.getElementById('cv-education').innerText = document.getElementById('input-education').value;
    }

    // ATS Matcher Algorithm
    function analyzeATS() {
      const jobDesc = document.getElementById('job-desc').value.toLowerCase();
      if (!jobDesc.trim()) {
        alert('Please paste a job description first.');
        return;
      }

      // Collect all text from CV
      const cvContent = (
        document.getElementById('input-summary').value + ' ' +
        document.getElementById('input-skills').value + ' ' +
        document.getElementById('input-experience').value + ' ' +
        document.getElementById('input-education').value
      ).toLowerCase();

      // Extract significant keywords (filtering out stop-words)
      const stopWords = new Set(['and', 'or', 'the', 'in', 'with', 'to', 'for', 'a', 'an', 'of', 'at', 'by', 'on', 'is', 'are', 'as', 'our', 'you', 'will']);
      const rawWords = jobDesc.match(/\b[a-zA-Z]{3,}\b/g) || [];
      const keywords = Array.from(new Set(rawWords.filter(w => !stopWords.has(w))));

      if (keywords.length === 0) return;

      let matched = [];
      let missing = [];

      keywords.forEach(kw => {
        if (cvContent.includes(kw)) {
          matched.push(kw);
        } else {
          missing.push(kw);
        }
      });

      const score = Math.round((matched.length / keywords.length) * 100);

      // Display results
      const resultsDiv = document.getElementById('ats-results');
      const scoreBar = document.getElementById('score-bar');
      const scoreText = document.getElementById('score-text');
      const missingText = document.getElementById('missing-keywords');

      resultsDiv.classList.remove('hidden');
      scoreText.innerText = `${score}% (${matched.length}/${keywords.length} keywords matched)`;
      scoreBar.style.width = `${score}%`;

      if (score < 50) {
        scoreBar.className = 'h-2.5 rounded-full bg-red-500';
      } else if (score < 75) {
        scoreBar.className = 'h-2.5 rounded-full bg-amber-500';
      } else {
        scoreBar.className = 'h-2.5 rounded-full bg-emerald-500';
      }

      missingText.innerHTML = `<strong>Missing keywords to consider adding:</strong> ` + 
        (missing.slice(0, 15).join(', ') || 'None! Great match.');
    }

    // PDF Download Engine
    function downloadPDF() {
      const element = document.getElementById('cv-preview');
      const opt = {
        margin:       [10, 10, 10, 10], // standard mm margins
        filename:     `${document.getElementById('input-name').value.replace(/\s+/g, '_')}_CV.pdf`,
        image:        { type: 'jpeg', quality: 0.98 },
        html2canvas:  { scale: 2, useCORS: true },
        jsPDF:        { unit: 'mm', format: 'a4', orientation: 'portrait' }
      };
      html2pdf().set(opt).from(element).save();
    }

    // Initialize content on load
    window.onload = updateCV;
  </script>
</body>
</html>
