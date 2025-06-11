<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>D. null - Terminal Access</title>
<style>
  /* Reset and Base */
  * {
    box-sizing: border-box;
  }
  body, html {
    margin: 0; padding: 0; height: 100%;
    background: #000000;
    color: #00ff00;
    font-family: 'Consolas', 'Courier New', monospace;
    font-size: 1.15rem;
    line-height: 1.4;
    user-select: none;
    overflow: hidden;
  }
  #app {
    display: flex;
    flex-direction: column;
    height: 100%;
    padding: 16px;
    position: relative;
    overflow: hidden;
  }

  /* Matrix style letter streams background */
  #matrix {
    position: fixed;
    top: 0; left: 0; width: 100vw; height: 100vh;
    pointer-events: none;
    z-index: 0;
    opacity: 0.1;
  }

  canvas {
    display: block;
  }

  /* Terminal container styles */
  #terminal {
    background: transparent;
    border: 2px solid #008000;
    border-radius: 4px;
    box-shadow:
      0 0 10px #00ff00,
      inset 0 0 20px #00bb00;
    flex: 1;
    padding: 20px;
    overflow-y: auto;
    white-space: pre-wrap;
    outline: none;
    position: relative;
    z-index: 1;
  }

  /* Scrollbar in green */
  #terminal::-webkit-scrollbar {
    width: 8px;
  }
  #terminal::-webkit-scrollbar-track {
    background: #001100;
  }
  #terminal::-webkit-scrollbar-thumb {
    background: #00bb00;
  }

  /* Prompt line styles */
  #prompt-line {
    display: flex;
    align-items: center;
    gap: 8px;
    font-weight: bold;
  }

  #prompt-text {
    flex-shrink: 0;
    user-select: text;
  }

  #password-input {
    background: none;
    border: none;
    color: #00ff00;
    font-family: inherit;
    font-size: inherit;
    outline: none;
    caret-color: #00ff00;
    user-select: text;
    flex-grow: 1;
    padding: 0;
    margin: 0;
  }

  /* Blinking cursor after input */
  #prompt-line.focused #password-input {
    caret-color: #00ff00;
  }

  /* Caret animation on after element for blinking block cursor */
  .cursor {
    display: inline-block;
    background-color: #00ff00;
    width: 8px;
    height: 1.2em;
    margin-left: 2px;
    animation: blink 1s steps(2, start) infinite;
    vertical-align: bottom;
  }
  @keyframes blink {
    0%, 50% { opacity: 1; }
    50.1%, 100% { opacity: 0; }
  }

  /* Typed output lines */
  .output-line {
    white-space: pre-wrap;
    user-select: text;
  }

  /* Footer styled like terminal status bar */
  footer {
    padding: 8px 16px;
    font-family: monospace;
    font-size: 0.9rem;
    color: #007700;
    text-shadow:
      0 0 10px #004400;
    background-color: #001100;
    border-top: 1px solid #004400;
    user-select: none;
    flex-shrink: 0;
  }

  /* Responsive */
  @media (max-width: 480px) {
    body, html {
      font-size: 1rem;
    }
    #terminal {
      padding: 12px;
    }
  }
</style>
</head>
<body>
<div id="matrix"></div>
<div id="app" role="application" aria-label="Terminal access for D. null website module">
  <div id="terminal" role="document" tabindex="0" aria-live="polite" aria-atomic="false" aria-relevant="additions">
    <!-- Terminal output lines will be appended here -->
  </div>
  <form id="password-form" role="form" aria-label="Password prompt" autocomplete="off" spellcheck="false" novalidate style="margin-top:8px;">
    <label for="password-input" class="visually-hidden">Password:</label>
    <div id="prompt-line" tabindex="0" aria-live="off" aria-atomic="false">
      <span id="prompt-text" aria-hidden="true">C:\Users\D.null&gt; </span>
      <input id="password-input" type="password" aria-describedby="prompt-instructions" autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false" autofocus />
      <span class="cursor" aria-hidden="true"></span>
    </div>
    <div id="prompt-instructions" style="position:absolute;left:-9999px;">Enter password to access the website</div>
  </form>
</div>
<footer aria-hidden="true" role="contentinfo">
  D. null Terminal Access © 2025 | Ver 1.0.0
</footer>

<script>
  // Matrix background effect
  (function(){
    const canvas = document.createElement('canvas');
    canvas.style.position = 'fixed';
    canvas.style.top = 0; canvas.style.left = 0;
    canvas.style.width = '100vw';
    canvas.style.height = '100vh';
    canvas.style.zIndex = 0;
    canvas.style.pointerEvents = 'none';
    document.getElementById('matrix').appendChild(canvas);
    const ctx = canvas.getContext('2d');
    let width, height;
    function resize() {
      width = canvas.width = window.innerWidth;
      height = canvas.height = window.innerHeight;
    }
    resize();
    window.addEventListener('resize', resize);

    const letters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789@#$%^&*()*&^%+-/~{[|`]}';
    const fontSize = 16;
    const columns = Math.floor(width / fontSize);
    let drops = new Array(columns).fill(1);

    function draw() {
      ctx.fillStyle = 'rgba(0, 0, 0, 0.1)';
      ctx.fillRect(0, 0, width, height);

      ctx.fillStyle = '#00ff00';
      ctx.font = fontSize + 'px monospace';

      for(let i = 0; i < drops.length; i++) {
        const text = letters.charAt(Math.floor(Math.random() * letters.length));
        ctx.fillText(text, i * fontSize, drops[i] * fontSize);

        if(drops[i] * fontSize > height && Math.random() > 0.975) {
          drops[i] = 0;
        }
        drops[i]++;
      }
    }
    setInterval(draw, 40);
  })();

  // Terminal typing simulation with password check
  (function(){
    const form = document.getElementById('password-form');
    const input = document.getElementById('password-input');
    const terminal = document.getElementById('terminal');
    const promptLine = document.getElementById('prompt-line');
    let isUnlocked = false;
    const correctPassword = 'D.null_D.null';

    // To keep tab focus on input
    promptLine.classList.add('focused');
    input.focus();

    // Utility: Create output line
    function addLine(text = '', className = '') {
      const line = document.createElement('div');
      line.textContent = text;
      if(className) line.className = className;
      terminal.appendChild(line);
      terminal.scrollTop = terminal.scrollHeight;
      return line;
    }

    // Typewriter effect for output lines
    function typeText(text, delay = 30) {
      return new Promise(resolve => {
        let i = 0;
        const line = document.createElement('div');
        line.className = 'output-line';
        terminal.appendChild(line);
        terminal.scrollTop = terminal.scrollHeight;

        function type() {
          if(i < text.length) {
            line.textContent += text.charAt(i);
            i++;
            terminal.scrollTop = terminal.scrollHeight;
            setTimeout(type, delay + Math.random() * 40);
          } else {
            resolve();
          }
        }
        type();
      });
    }

    // Clear terminal screen content except prompt form
    function clearTerminalOutput() {
      terminal.innerHTML = '';
    }

    // Show error message in terminal
    function showAccessDenied() {
      addLine('Access Denied. Incorrect password.');
    }
    // Initial greeting lines
    async function showBootupSequence() {
      clearTerminalOutput();
      await typeText('Initializing D.null Secure Terminal Access...');
      await typeText('Loading modules: [█████████████] 100%');
      await typeText('System check: All systems nominal.');
      await typeText('Enter password to continue.');
    }

    // Show message after unlock
    async function showWelcomeMessage() {
      clearTerminalOutput();
      await typeText('Access Granted.\n');
      await typeText('This Website Belongs To D. null\n');
      await typeText('\n[WARNING] Unauthorized access will be traced.\n');
      await typeText('--------------------------------------------------\n');
      await typeText('Welcome back, Commander.\n');
    }

    // On form submit
    form.addEventListener('submit', async function(e){
      e.preventDefault();
      if(isUnlocked) return;
      const val = input.value.trim();
      input.value = '';
      promptLine.classList.remove('focused');
      input.disabled = true;

      if(val === correctPassword) {
        isUnlocked = true;
        await showWelcomeMessage();
        form.style.display = 'none';
      } else {
        showAccessDenied();
        input.disabled = false;
        promptLine.classList.add('focused');
        input.focus();
      }
    });

    // Trap focus inside prompt
    document.addEventListener('keydown', (e) => {
      if(!isUnlocked && e.key === 'Tab') {
        e.preventDefault();
        input.focus();
      }
    });

    // Run bootup sequence on page load
    showBootupSequence();

  })();
</script>
</body>
</html>

