<!DOCTYPE html>
<html lang="pl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>CV Service - Tworzenie CV</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0; padding: 20px;
      background: #f9f9f9;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 20px;
    }
    h1 {
      width: 100%;
      text-align: center;
      color: #222;
      margin-bottom: 10px;
    }
    form {
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px #ccc;
      max-width: 450px;
      flex-grow: 1;
      min-width: 300px;
      box-sizing: border-box;
    }
    label {
      display: block;
      margin-top: 12px;
      font-weight: bold;
      color: #444;
    }
    input[type="text"],
    input[type="email"],
    textarea {
      width: 100%;
      padding: 8px;
      margin-top: 5px;
      border-radius: 4px;
      border: 1px solid #ccc;
      font-size: 1em;
      box-sizing: border-box;
      resize: vertical;
    }
    textarea {
      min-height: 60px;
      max-height: 120px;
    }
    button {
      margin-top: 20px;
      padding: 12px;
      background: #007bff;
      border: none;
      color: white;
      font-size: 1em;
      border-radius: 5px;
      cursor: pointer;
      width: 100%;
    }
    button:hover {
      background: #0056b3;
    }

    /* Podgląd CV */
    .cv-preview {
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px #ccc;
      max-width: 450px;
      flex-grow: 1;
      min-width: 300px;
      box-sizing: border-box;
      color: #222;
      overflow-wrap: break-word;
    }
    .cv-preview h2 {
      margin-top: 0;
      border-bottom: 3px solid #007bff;
      padding-bottom: 6px;
      color: #007bff;
    }
    .cv-section {
      margin-top: 15px;
    }
    .cv-section strong {
      display: inline-block;
      width: 140px;
      color: #555;
    }
    .skills-list, .languages-list {
      list-style: inside disc;
      padding-left: 0;
      margin-top: 5px;
    }

    @media(max-width: 900px) {
      body {
        flex-direction: column;
        align-items: center;
      }
      form, .cv-preview {
        max-width: 100%;
        min-width: auto;
      }
    }
  </style>
</head>
<body>

  <h1>CV Service - Tworzenie CV</h1>

  <form id="cvForm">
    <label for="firstName">Imię:</label>
    <input type="text" id="firstName" name="firstName" required />

    <label for="lastName">Nazwisko:</label>
    <input type="text" id="lastName" name="lastName" required />

    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required />

    <label for="experience">Doświadczenie zawodowe:</label>
    <textarea id="experience" name="experience" placeholder="Opisz swoje doświadczenie" required></textarea>

    <label for="education">Edukacja:</label>
    <textarea id="education" name="education" placeholder="Twoja edukacja" required></textarea>

    <label for="skills">Umiejętności (oddziel przecinkami):</label>
    <input type="text" id="skills" name="skills" placeholder="np. JavaScript, Python, Zarządzanie" required />

    <label for="languages">Języki (oddziel przecinkami):</label>
    <input type="text" id="languages" name="languages" placeholder="np. Polski, Angielski, Niemiecki" required />

    <button type="submit">Generuj i Zapisz CV</button>
    <button type="button" id="downloadPdfBtn" style="margin-top:10px; background:#28a745;">Pobierz CV jako PDF</button>
    <button type="button" id="loadBtn" style="margin-top:10px; background:#6c757d;">Wczytaj zapisane dane</button>
  </form>

  <div class="cv-preview" id="cvPreview">
    <h2>Podgląd CV</h2>
    <div class="cv-section"><strong>Imię i nazwisko:</strong> <span id="previewName">---</span></div>
    <div class="cv-section"><strong>Email:</strong> <span id="previewEmail">---</span></div>
    <div class="cv-section"><strong>Doświadczenie:</strong> <p id="previewExperience">---</p></div>
    <div class="cv-section"><strong>Edukacja:</strong> <p id="previewEducation">---</p></div>
    <div class="cv-section"><strong>Umiejętności:</strong>
      <ul id="previewSkills" class="skills-list"></ul>
    </div>
    <div class="cv-section"><strong>Języki:</strong>
      <ul id="previewLanguages" class="languages-list"></ul>
    </div>
  </div>

  <!-- Biblioteka jsPDF do eksportu PDF -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

  <script>
    const { jsPDF } = window.jspdf;

    const form = document.getElementById('cvForm');
    const previewName = document.getElementById('previewName');
    const previewEmail = document.getElementById('previewEmail');
    const previewExperience = document.getElementById('previewExperience');
    const previewEducation = document.getElementById('previewEducation');
    const previewSkills = document.getElementById('previewSkills');
    const previewLanguages = document.getElementById('previewLanguages');

    const downloadPdfBtn = document.getElementById('downloadPdfBtn');
    const loadBtn = document.getElementById('loadBtn');

    // Aktualizacja podglądu
    function updatePreview() {
      const firstName = form.firstName.value.trim();
      const lastName = form.lastName.value.trim();
      const email = form.email.value.trim();
      const experience = form.experience.value.trim();
      const education = form.education.value.trim();
      const skills = form.skills.value.trim();
      const languages = form.languages.value.trim();

      previewName.textContent = firstName && lastName ? firstName + ' ' + lastName : '---';
      previewEmail.textContent = email || '---';
      previewExperience.innerHTML = experience ? experience.replace(/\n/g, '<br>') : '---';
      previewEducation.innerHTML = education ? education.replace(/\n/g, '<br>') : '---';

      // Skills list
      previewSkills.innerHTML = '';
      if (skills) {
        skills.split(',').map(s => s.trim()).filter(s => s).forEach(skill => {
          const li = document.createElement('li');
          li.textContent = skill;
          previewSkills.appendChild(li);
        });
      } else {
        previewSkills.innerHTML = '<li>---</li>';
      }

      // Languages list
      previewLanguages.innerHTML = '';
      if (languages) {
        languages.split(',').map(l => l.trim()).filter(l => l).forEach(lang => {
          const li = document.createElement('li');
          li.textContent = lang;
          previewLanguages.appendChild(li);
        });
      } else {
        previewLanguages.innerHTML = '<li>---</li>';
      }
    }

    // Na żywo aktualizuj podgląd przy zmianach w formularzu
    form.addEventListener('input', updatePreview);

    // Obsługa zapisu i generowania CV po submit
    form.addEventListener('submit', e => {
      e.preventDefault();

      // Walidacja
      if (!form.firstName.value.trim() || !form.lastName.value.trim() || !form.email.value.trim() ||
          !form.experience.value.trim() || !form.education.value.trim() || !form.skills.value.trim() ||
          !form.languages.value.trim()) {
        alert('Proszę wypełnić wszystkie pola.');
        return;
      }

      // Zapis do localStorage
      const cvData = {
        firstName: form.firstName.value.trim(),
        lastName: form.lastName.value.trim(),
        email: form.email.value.trim(),
        experience: form.experience.value.trim(),
        education: form.education.value.trim(),
        skills: form.skills.value.trim(),
        languages: form.languages.value.trim()
      };
      localStorage.setItem('cvData', JSON.stringify(cvData));
      alert('Dane CV zostały zapisane lokalnie.');

      updatePreview();
    });

    // Pobierz dane z localStorage i wczytaj do formularza
    loadBtn.addEventListener('click', () => {
      const savedData = localStorage.getItem('cvData');
      if (!savedData) {
        alert('Brak zapisanych danych.');
        return;
      }
      const cvData = JSON.parse(savedData);
      form.firstName.value = cvData.firstName || '';
      form.lastName.value = cvData.lastName || '';
      form.email.value = cvData.email || '';
      form.experience.value = cvData.experience || '';
      form.education.value = cvData.education || '';
      form.skills.value = cvData.skills || '';
      form.languages.value = cvData.languages || '';

      updatePreview();
      alert('Dane zostały wczytane.');
    });

    // Eksport do PDF
    downloadPdfBtn.addEventListener('click', () => {
      const doc = new jsPDF();

      const marginLeft = 15;
      let y = 20;

      doc.setFontSize(22);
      doc.setTextColor('#007bff');
      doc.text("Curriculum Vitae", 105, y, null, null, 'center');
      y += 15;

      doc.setFontSize(14);
      doc.setTextColor('#000');
      doc.text(`Imię i nazwisko: ${form.firstName.value.trim()} ${form.lastName.value.trim()}`, marginLeft, y);
      y += 10;
      doc.text(`Email: ${form.email.value.trim()}`, marginLeft, y);
      y += 15;

      doc.setFontSize(16);
      doc.setTextColor('#007bff');
      doc.text('Doświadczenie:', marginLeft, y);
      y += 10;
      doc.setFontSize(12);
      const expLines = doc.splitTextToSize(form.experience.value.trim(), 180);
      doc.text(expLines, marginLeft, y);
      y += expLines.length * 7 + 5;

      doc.setFontSize(16);
      doc.setTextColor('#007bff');
      doc.text('Edukacja:', marginLeft, y);
      y += 10;
      doc.setFontSize(12);
      const eduLines = doc.splitTextToSize(form.education.value.trim(), 180);
      doc.text(eduLines, marginLeft, y);
      y += eduLines.length * 7 + 5;

      doc.setFontSize(16);
      doc.setTextColor('#007bff');
      doc.text('Umiejętności:', marginLeft, y);
      y += 10;
      doc.setFontSize(12);
      const skillsArr = form.skills.value.trim().split(',').map(s => s.trim()).filter(s => s);
      skillsArr.forEach(skill => {
        doc.text(`- ${skill}`, marginLeft + 5, y);
        y += 7;
      });
      y += 5;

      doc.setFontSize(16);
      doc.setTextColor('#007bff');
      doc.text('Języki:', marginLeft, y);
      y += 10;
      doc.setFontSize(12);
      const langsArr = form.languages.value.trim().split(',').map(l => l.trim()).filter(l => l);
      langsArr.forEach(lang => {
        doc.text(`- ${lang}`, marginLeft + 5, y);
        y += 7;
      });

      doc.save('CV.pdf');
    });

    // Inicjalizacja podglądu przy załadowaniu strony
    updatePreview();

  </script>

</body>
</html>
