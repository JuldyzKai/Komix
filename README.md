
<html lang="kk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Кітап Флип-Эффектісі</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #f5f5f5;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
            overflow-x: hidden;
        }
        
        .book-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 100%;
            max-width: 1200px;
        }
        
        .book {
            position: relative;
            width: 400px;
            height: 620px;
            perspective: 2000px;
            margin-bottom: 20px;
            border: 3px solid #8B4513;
            border-radius: 5px 15px 15px 5px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
        }
        
        .cover {
            position: absolute;
            width: 100%;
            height: 100%;
            background: url('артқы фонда дала тау.png') center/cover;
            color: white;
            border-radius: 5px 15px 15px 5px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
            transform-origin: left;
            transform-style: preserve-3d;
            transition: transform 1s ease-in-out;
            z-index: 100;
            display: flex;
            flex-direction: column;
            justify-content: flex-end;
            align-items: center;
            text-align: center;
            padding: 20px;
            box-sizing: border-box;
        }
        
        .cover h1 {
            margin-bottom: 10px;
        }
        
        .cover p {
            margin: 5px 0;
        }
        
        .cover.open {
            transform: rotateY(-160deg);
        }
        
        .pages-container {
            position: absolute;
            width: 100%;
            height: 100%;
            display: none;
            background: #fff;
            border-radius: 5px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
            overflow: hidden;
            box-sizing: border-box;
        }
        
        .pages-container.open {
            display: block;
        }
        
        .page {
            position: absolute;
            width: 100%;
            height: 100%;
            padding: 0;
            box-sizing: border-box;
            overflow: hidden;
            background: #fff;
            transition: transform 0.7s ease-in-out;
            backface-visibility: hidden;
            transform-style: preserve-3d;
            transform-origin: left;
        }
        
        .page img {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }
        
        .page.flipping {
            transform: rotateY(-160deg);
            z-index: 3;
        }
        
        .page-number {
            position: absolute;
            bottom: 10px;
            right: 20px;
            font-size: 14px;
            color: #888;
            background: rgba(255,255,255,0.7);
            padding: 2px 5px;
            border-radius: 3px;
        }
        
        .controls {
            display: flex;
            justify-content: center;
            gap: 15px;
            margin-top: 20px;
            width: 100%;
        }
        
        .control-btn {
            padding: 10px 20px;
            background: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            transition: background 0.3s;
        }
        
        .control-btn:hover {
            background: #45a049;
        }
        
        .control-btn:disabled {
            background: #cccccc;
            cursor: not-allowed;
        }
        
        .fullscreen-btn {
            position: absolute;
            top: 10px;
            right: 10px;
            padding: 5px 10px;
            background: rgba(0,0,0,0.5);
            color: white;
            border: none;
            border-radius: 3px;
            cursor: pointer;
            z-index: 200;
        }
        
        @media (max-width: 768px) {
            .book {
                height: 400px;
            }
        }
        
        /* Музыканы жасыру */
        #audioPlayer {
            display: none;
        }
    </style>
</head>
<body>
    <div class="book-container">
        <div class="book">
            <div class="cover" id="cover">
                <div>
                    <h1></h1>
                    <p> Қайназарова Жулдыз Есімханқызы</p>
                    <p>Алматы-2025</p>
                </div>
            </div>
            
            <div class="pages-container" id="pagesContainer">
                <!-- Беттер автоматты түрде генерацияланады -->
            </div>
            
            <button class="fullscreen-btn" id="fullscreenBtn">Экран</button>
        </div>
        
        <div class="controls">
            <button class="control-btn" id="prevBtn"> ←  </button>
            <button class="control-btn" id="toggleBtn">Жабу</button>
            <button class="control-btn" id="nextBtn"> → </button>
        </div>
    </div>

    <!-- Музыканы жасыру -->
    <audio id="audioPlayer" autoplay loop>
        <source src="Нұрғиса Тілендиев - Жекпе-жек.mp3" type="audio/mpeg">
        Сіздің браузеріңіз аудио элементін қолдамайды.
    </audio>

    <script>
        // Айнымалылар
        const toggleBtn = document.getElementById('toggleBtn');
        const prevBtn = document.getElementById('prevBtn');
        const nextBtn = document.getElementById('nextBtn');
        const fullscreenBtn = document.getElementById('fullscreenBtn');
        const cover = document.getElementById('cover');
        const pagesContainer = document.getElementById('pagesContainer');
        const audioPlayer = document.getElementById('audioPlayer');
        
        let isOpen = true;
        let currentPage = 0;
        const totalPages = 28;
        let pages = [];
        let isFlipping = false;
        
        // Суреттердің тізімі (өз суреттеріңізді қойыңыз)
        const pageImages = [
            '1.jpg', '2.jpg','А мен Т.jpg', '3.jpg', '5-бөл.jpg', 'Шеге мен төлеген.jpg','6-бөлім_ Ақжайыққа ж.png',
            '7.png','Жібек пен Т.jpg', 'комикс сөзін біркелк.png', 'Жібек пен Сансызбайд.png', '9.png', 'Бекежанның қызғанышы.png',
            '10. .png', '4.jpg', 'Төлегеннің Сансызбай.png', 'Төлегеннің жасырын к.png', 'ойбай сұлуларды әдем.png',
            'Add comic-style text.png', 'ББЕ.png', 'Бек.png','Бекежан.jpg', 'Төлеген мен Бекежанд.png', '19-бөлім_ Қанға – қа.png','сансызбайдың түрі тө.png', 
            'ҚызЖ.jpg', 'Қалыңд.jpg',  'Жібек пен Сансызбай.jpg'
        ];
        
        // Беттерді генерациялау
        function generatePages() {
            pagesContainer.innerHTML = '';
            pages = [];
            
            for (let i = 0; i < totalPages; i++) {
                const page = document.createElement('div');
                page.className = 'page';
                page.innerHTML = `
                    <img src="${pageImages[i]}" alt="Сурет ${i+1}">
                    <div class="page-number">${i + 1}/${totalPages}</div>
                `;
                page.dataset.pageIndex = i;
                pagesContainer.appendChild(page);
                pages.push(page);
                
                // Бастапқыда барлық беттерді жасыру
                if (i > 0) {
                    page.style.display = 'none';
                }
            }
        }
        
        // Бетті аудару анимациясы
        function flipPage(direction) {
            if (isFlipping) return;
            isFlipping = true;
            
            const nextPageIndex = direction === 'next' ? currentPage + 1 : currentPage - 1;
            
            if (nextPageIndex < 0 || nextPageIndex >= totalPages) {
                isFlipping = false;
                return;
            }
            
            const currentPageEl = pages[currentPage];
            const nextPageEl = pages[nextPageIndex];
            
            // Келесі бетті көрсету
            nextPageEl.style.display = 'block';
            
            if (direction === 'next') {
                currentPageEl.classList.add('flipping');
            } else {
                nextPageEl.classList.add('flipping');
            }
            
            // Анимация аяқталғаннан кейін
            setTimeout(() => {
                currentPageEl.style.display = 'none';
                currentPageEl.classList.remove('flipping');
                nextPageEl.classList.remove('flipping');
                
                currentPage = nextPageIndex;
                updateButtons();
                isFlipping = false;
            }, 700);
        }
        
        // Батырмаларды жаңарту
        function updateButtons() {
            prevBtn.disabled = currentPage === 0;
            nextBtn.disabled = currentPage === totalPages - 1;
        }
        
        // Кітапты ашу/жабу
        function toggleBook() {
            isOpen = !isOpen;
            
            if (isOpen) {
                cover.classList.add('open');
                pagesContainer.classList.add('open');
                toggleBtn.textContent = 'жабу';
            } else {
                cover.classList.remove('open');
                pagesContainer.classList.remove('open');
                toggleBtn.textContent = 'ашу';
            }
        }
        
        // Алдыңғы бетке өту
        function goToPrevPage() {
            flipPage('prev');
        }
        
        // Келесі бетке өту
        function goToNextPage() {
            flipPage('next');
        }
        
        // Толық экран режимі
        function toggleFullscreen() {
            if (!document.fullscreenElement) {
                document.documentElement.requestFullscreen().catch(err => {
                    console.log(`Толық экран қатесі: ${err.message}`);
                });
                fullscreenBtn.textContent = 'Толық экраннан шығу';
            } else {
                if (document.exitFullscreen) {
                    document.exitFullscreen();
                    fullscreenBtn.textContent = 'Толық экран';
                }
            }
        }
        
        // Іс-шараларды байлау
        toggleBtn.addEventListener('click', toggleBook);
        prevBtn.addEventListener('click', goToPrevPage);
        nextBtn.addEventListener('click', goToNextPage);
        fullscreenBtn.addEventListener('click', toggleFullscreen);
        
        // Бастапқы инициализация
        generatePages();
        toggleBook(); // Бастапқыда ашық күйде
        updateButtons();
        
        // Беттерді ауыстыру үшін пернелерді басқару
        document.addEventListener('keydown', (e) => {
            if (isOpen && !isFlipping) {
                if (e.key === 'ArrowLeft') {
                    goToPrevPage();
                } else if (e.key === 'ArrowRight') {
                    goToNextPage();
                }
            }
        });
        
        // Беттерді шертіп ауыстыру
        let touchStartX = 0;
        let touchEndX = 0;
        
        pagesContainer.addEventListener('touchstart', (e) => {
            touchStartX = e.changedTouches[0].screenX;
        }, false);
        
        pagesContainer.addEventListener('touchend', (e) => {
            touchEndX = e.changedTouches[0].screenX;
            handleSwipe();
        }, false);
        
        function handleSwipe() {
            if (Math.abs(touchEndX - touchStartX) < 50) return; // Кішкене қозғалыстарды елемеу
            
            if (touchEndX < touchStartX && !isFlipping) {
                // Оңға сырғытқанда (сол жаққа)
                goToNextPage();
            } else if (touchEndX > touchStartX && !isFlipping) {
                // Солға сырғытқанда (оң жаққа)
                goToPrevPage();
            }
        }
        
        // Музыканы автоматты түрде ойнату
        document.addEventListener('click', function() {
            
            audioPlayer.play().catch(e => console.log('Автоойнату бұғатталған:', e));
        }, { once: true });
    </script>
</body>
</html>
