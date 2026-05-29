// =====================
// STATE & CONFIG
// =====================
let currentType = 'ps';
let currentSubtype = 'ps5';
let gamepadIndex = null;
let rafId = null;
let lastTimestamp = 0;
let frameCount = 0;
let fps = 0;
let manualMode = false;

// إحداثيات العصا الحالية (اليسرى واليمنى)
let stickStates = {
  left: { x: 0, y: 0, isDragging: false },
  right: { x: 0, y: 0, isDragging: false }
};

const settings = {
  deadzoneLeft: 8,
  deadzoneRight: 8,
  sensLeft: 100,
  sensRight: 100,
  trigDZ: 5
};

// ... (تُترك مصفوفات الـ controllers و BUTTON_MAP و AXES كما هي في كودك القديم) ...

// =====================
// دالة التحديث المستمر الذكية (Game Loop)
// =====================
function gameLoop(ts) {
  rafId = requestAnimationFrame(gameLoop);
  frameCount++;
  
  if (ts - lastTimestamp >= 1000) {
    fps = frameCount;
    frameCount = 0;
    lastTimestamp = ts;
    document.getElementById('frameText').textContent = fps + ' fps';
  }

  // قراءة بيانات يد التحكم الحقيقية إن وجدت
  const gamepads = navigator.getGamepads();
  const gp = gamepads[gamepadIndex];

  if (gp) {
    manualMode = false; // إلغاء الوضع اليدوي فوراً عند رصد يد حقيقية
    if (gp.battery) {
      document.getElementById('battText').textContent = `البطارية: ${Math.round(gp.battery.level * 100)}%`;
    }
    // قراءة المحاور الحقيقية من اليد
    stickStates.left.x = gp.axes[0];
    stickStates.left.y = gp.axes[1];
    stickStates.right.x = gp.axes[2];
    stickStates.right.y = gp.axes[3];
    
    updateTriggerVisual(gp);
    updateButtonsVisual(gp);
  }

  // معالجة وتطبيق المعايرة (سواء لليد الحقيقية أو السحب بالماوس)
  processAndRenderStick('left', 'stickLeft', 'coordsLeft');
  processAndRenderStick('right', 'stickRight', 'coordsRight');
}

// =====================
// المحرك الفيزيائي للمعايرة وتطبيق المنحنيات
// =====================
function processAndRenderStick(side, stickDotId, coordsId) {
  let state = stickStates[side];
  let rawX = state.x;
  let rawY = state.y;
  
  let dz = (side === 'left' ? settings.deadzoneLeft : settings.deadzoneRight) / 100;
  let sens = (side === 'left' ? settings.sensLeft : settings.sensRight) / 100;
  let curve = document.getElementById('curveType').value;

  // 1. حساب القوة المتجهة (Magnitude) لعصا التحكم
  let magnitude = Math.sqrt(rawX * rawX + rawY * rawY);
  let calX = 0;
  let calY = 0;

  if (magnitude > dz) {
    // تطبيع القيمة بعد طرح المنطقة الميتة (Deadzone Normalized)
    let normalizedMag = (magnitude - dz) / (1 - dz);
    
    // 2. تطبيق منحنيات الحساسية المتقدمة (Calibration Curves)
    if (curve === 'expo') {
      normalizedMag = Math.pow(normalizedMag, 2.5); // أسي سريع
    } else if (curve === 'scurve') {
      normalizedMag = normalizedMag * normalizedMag * (3 - 2 * normalizedMag); // ناعم في المنتصف وحاد في الأطراف
    } else if (curve === 'steep') {
      normalizedMag = Math.pow(normalizedMag, 4); // حاد جداً للاستجابة اللحظية
    }
    
    // إعادة ضرب القوة المحدثة بمعامل الحساسية والاتجاه الزاوي
    let finalMag = normalizedMag * sens;
    finalMag = Math.min(1, finalMag); // سقف القوة الأقصى هو 1.0
    
    calX = (rawX / magnitude) * finalMag;
    calY = (rawY / magnitude) * finalMag;
  }

  // 3. التحديث البصري على الشاشة (الـ DOM)
  const dot = document.getElementById(stickDotId);
  if (dot) {
    // تحويل النطاق من (-1 إلى 1) إلى بكسلات داخل الدائرة (نصف القطر 65 بكسل وعرض النقطة 18 بكسل)
    let parentSize = 130;
    let center = parentSize / 2;
    let radius = center - 9; // نطاق الحركة الأقصى المسموح به للنقطة
    
    let pixelX = center + (calX * radius);
    let pixelY = center + (calY * radius);
    
    dot.style.left = pixelX + 'px';
    dot.style.top = pixelY + 'px';
  }

  document.getElementById(coordsId).textContent = `X: ${calX.toFixed(2)} | Y: ${calY.toFixed(2)}`;
}

// =====================
// تفعيل ميزة السحب التفاعلي بالماوس (Interactive Dragging)
// =====================
function setupSliderListeners() {
  ['Left', 'Right'].forEach(side => {
    const bg = document.getElementById(`stick${side}Bg`);
    const sKey = side.toLowerCase();
    
    const handleMove = (clientX, clientY) => {
      if (!stickStates[sKey].isDragging) return;
      const rect = bg.getBoundingClientRect();
      const centerX = rect.left + rect.width / 2;
      const centerY = rect.top + rect.height / 2;
      
      // حساب المسافة الفيزيائية من المركز ونمذجتها بين -1 و 1
      let rawX = (clientX - centerX) / (rect.width / 2);
      let rawY = (clientY - centerY) / (rect.height / 2);
      
      // حصر القيم داخل نطاق دائرى مثالي
      let mag = Math.sqrt(rawX*rawX + rawY*rawY);
      if (mag > 1) {
        rawX /= mag;
        rawY /= mag;
      }
      
      stickStates[sKey].x = rawX;
      stickStates[sKey].y = rawY;
    };

    // مستمعات الأحداث للماوس واللمس للهواتف
    bg.addEventListener('mousedown', (e) => { stickStates[sKey].isDragging = true; manualMode = true; handleMove(e.clientX, e.clientY); });
    window.addEventListener('mousemove', (e) => handleMove(e.clientX, e.clientY));
    window.addEventListener('mouseup', () => stickStates[sKey].isDragging = false);
    
    bg.addEventListener('touchstart', (e) => { stickStates[sKey].isDragging = true; manualMode = true; handleMove(e.touches[0].clientX, e.touches[0].clientY); }, {passive: true});
    window.addEventListener('touchmove', (e) => { if(stickStates[sKey].isDragging) handleMove(e.touches[0].clientX, e.touches[0].clientY); }, {passive: true});
    window.addEventListener('touchend', () => stickStates[sKey].isDragging = false);
  });
}

// ... (باقي الدوال المتواجدة لديك مثل تشغيل الأزرار وتحديث الشاشات تظل بدون تغيير) ...
