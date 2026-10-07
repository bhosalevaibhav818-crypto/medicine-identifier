const medicineCatalog = [
  {
    id: 'paracetamol',
    name: 'Paracetamol',
    type: 'Pain reliever / fever reducer',
    commonUse: 'Relieves pain and reduces fever.',
    whenToTake: 'Usually take after food or with water. Follow the label instructions.',
    dosage: 'Follow the dose printed on the pack or prescribed by a healthcare professional.',
    warnings: 'Do not exceed the maximum daily dose. Avoid alcohol and check with a doctor if you have liver disease.',
    synonyms: ['acetaminophen', 'calpol', 'crocin', 'panadol']
  },
  {
    id: 'amoxicillin',
    name: 'Amoxicillin',
    type: 'Antibiotic',
    commonUse: 'Treats bacterial infections such as throat infections, ear infections, and sinus infection.',
    whenToTake: 'Take as directed, often with food to reduce stomach upset.',
    dosage: 'Complete the full course unless a doctor tells you otherwise.',
    warnings: 'Do not use if allergic to penicillin. Some medicines can interact with it.',
    synonyms: ['amox', 'amoxycillin']
  },
  {
    id: 'cetirizine',
    name: 'Cetirizine',
    type: 'Antihistamine',
    commonUse: 'Helps relieve allergy symptoms such as sneezing, runny nose, and itching.',
    whenToTake: 'Usually taken once daily, ideally in the evening if it makes you drowsy.',
    dosage: 'Use the amount printed on the pack or as prescribed.',
    warnings: 'May cause drowsiness. Avoid driving if you feel sleepy.',
    synonyms: ['zyrtec', 'allercet']
  },
  {
    id: 'ibuprofen',
    name: 'Ibuprofen',
    type: 'Anti-inflammatory pain medicine',
    commonUse: 'Used for pain, fever, and inflammation.',
    whenToTake: 'Take with food or milk to reduce stomach irritation.',
    dosage: 'Do not exceed the recommended dose on the label.',
    warnings: 'May irritate the stomach. Avoid if you have ulcers, kidney issues, or are pregnant unless advised by a doctor.',
    synonyms: ['advil', 'brufen', 'nurofen']
  },
  {
    id: 'omeprazole',
    name: 'Omeprazole',
    type: 'Acid reducer / antacid medicine',
    commonUse: 'Reduces stomach acid and helps with reflux, ulcers, and indigestion.',
    whenToTake: 'Usually taken before breakfast or as directed on the pack.',
    dosage: 'Follow the pack instructions or your doctor’s advice.',
    warnings: 'It may interact with some medicines. Seek advice if symptoms persist.',
    synonyms: ['prilosec', 'losec']
  },
  {
    id: 'metformin',
    name: 'Metformin',
    type: 'Diabetes medicine',
    commonUse: 'Helps control blood sugar in type 2 diabetes.',
    whenToTake: 'Usually taken with meals to reduce stomach side effects.',
    dosage: 'Take exactly as prescribed by your doctor.',
    warnings: 'Do not take if you have severe kidney problems or are dehydrated. Inform your doctor before tests involving contrast dye.',
    synonyms: ['glucophage']
  },
  {
    id: 'loratadine',
    name: 'Loratadine',
    type: 'Non-drowsy antihistamine',
    commonUse: 'Useful for seasonal allergies and itchy eyes.',
    whenToTake: 'Take once daily, usually with or without food.',
    dosage: 'Follow the label directions.',
    warnings: 'Although less sedating, consult a doctor if you feel unusual symptoms.',
    synonyms: ['claritin']
  },
  {
    id: 'azithromycin',
    name: 'Azithromycin',
    type: 'Antibiotic',
    commonUse: 'Treats many bacterial infections, including chest and throat infections.',
    whenToTake: 'Usually taken once daily, with or without food as directed.',
    dosage: 'Complete the full course as prescribed.',
    warnings: 'Tell your doctor if you have heart rhythm problems or are taking other medicines that affect the heart.',
    synonyms: ['zithromax']
  }
];

const searchInput = document.getElementById('medicineSearch');
const searchButton = document.getElementById('searchButton');
const scanButton = document.getElementById('scanButton');
const scanInput = document.getElementById('scanInput');
const statusText = document.getElementById('statusText');
const medicineCard = document.getElementById('medicineCard');
const ocrResult = document.getElementById('ocrResult');
const ocrText = document.getElementById('ocrText');

function normalizeString(value = '') {
  return value
    .toLowerCase()
    .trim()
    .replace(/[^a-z0-9\s]/g, ' ')
    .replace(/\s+/g, ' ');
}

function findMedicineByQuery(query) {
  const normalized = normalizeString(query);

  if (!normalized) {
    return null;
  }

  return medicineCatalog.find((medicine) => {
    const haystacks = [
      medicine.name,
      medicine.type,
      ...medicine.synonyms
    ].map(normalizeString);

    return haystacks.some((text) => text.includes(normalized) || normalized.includes(text));
  }) || null;
}

function renderMedicine(medicine) {
  if (!medicine) {
    medicineCard.classList.add('empty');
    medicineCard.innerHTML = '<p>No matching medicine found. Try a different name.</p>';
    return;
  }

  medicineCard.classList.remove('empty');
  medicineCard.innerHTML = `
    <h3>${medicine.name}</h3>
    <div class="medicine-meta">
      <span class="meta-pill">${medicine.type}</span>
    </div>
    <div class="details-grid">
      <div class="detail-box">
        <h4>Common use</h4>
        <p>${medicine.commonUse}</p>
      </div>
      <div class="detail-box">
        <h4>When to take</h4>
        <p>${medicine.whenToTake}</p>
      </div>
      <div class="detail-box">
        <h4>Dosage</h4>
        <p>${medicine.dosage}</p>
      </div>
      <div class="detail-box">
        <h4>Warnings</h4>
        <p>${medicine.warnings}</p>
      </div>
    </div>
    <div class="notice">
      This app gives general medical information only and is not a substitute for a doctor or pharmacist’s advice.
    </div>
  `;
}

function handleSearch() {
  const query = searchInput.value.trim();
  if (!query) {
    statusText.textContent = 'Please type a medicine name to search.';
    return;
  }

  const found = findMedicineByQuery(query);

  if (!found) {
    statusText.textContent = `No exact match found for “${query}”. Try a common name or brand name.`;
    medicineCard.classList.add('empty');
    medicineCard.innerHTML = '<p>No matching medicine found. Try a different name.</p>';
    return;
  }

  statusText.textContent = `Showing details for ${found.name}.`;
  renderMedicine(found);
}

async function analyzeImage(file) {
  if (!file) {
    statusText.textContent = 'Please choose an image first.';
    return;
  }

  statusText.textContent = 'Reading medicine label...';

  try {
    const Tesseract = await import('tesseract.js');
    const result = await Tesseract.recognize(file, 'eng');
    const text = result.data.text || '';

    if (!text.trim()) {
      statusText.textContent = 'No clear text found in the image. Try a sharper image.';
      return;
    }

    ocrText.textContent = text.trim();
    ocrResult.classList.remove('hidden');

    const matchedMedicine = medicineCatalog.find((medicine) => {
      const matchText = [medicine.name, ...medicine.synonyms].join(' ').toLowerCase();
      return text.toLowerCase().includes(matchText) || matchText.includes(text.toLowerCase());
    });

    if (matchedMedicine) {
      statusText.textContent = `Detected medicine: ${matchedMedicine.name}.`;
      renderMedicine(matchedMedicine);
      return;
    }

    const lines = text
      .split(/\n|\s{2,}/)
      .map((line) => line.trim())
      .filter(Boolean);

    for (const line of lines) {
      const candidate = findMedicineByQuery(line);
      if (candidate) {
        statusText.textContent = `Detected medicine from the scanned label: ${candidate.name}.`;
        renderMedicine(candidate);
        return;
      }
    }

    statusText.textContent = 'We found text, but it did not match a known medicine in this demo database.';
    renderMedicine(null);
  } catch (error) {
    console.error(error);
    statusText.textContent = 'Image analysis failed. Please try another image.';
  }
}

searchButton.addEventListener('click', handleSearch);
searchInput.addEventListener('keydown', (event) => {
  if (event.key === 'Enter') {
    handleSearch();
  }
});

scanButton.addEventListener('click', () => {
  if (!scanInput.files || scanInput.files.length === 0) {
    statusText.textContent = 'Choose an image of a medicine label first.';
    return;
  }

  analyzeImage(scanInput.files[0]);
});

scanInput.addEventListener('change', () => {
  if (scanInput.files && scanInput.files.length > 0) {
    analyzeImage(scanInput.files[0]);
  }
});

renderMedicine(null);
