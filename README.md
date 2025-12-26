<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flip Clock</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');

        body {
            font-family: 'Inter', sans-serif;
            overflow: hidden;
            margin: 0;
            padding: 0;
        }

        .perspective-container {
            perspective: 1000px;
        }

        .upper-card {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 50%;
            overflow: hidden;
            z-index: 10;
            backface-visibility: hidden;
            transform-origin: bottom;
            border-top-left-radius: 1.5rem; 
            border-top-right-radius: 1.5rem;
        }

        .lower-card {
            position: absolute;
            top: 50%;
            left: 0;
            width: 100%;
            height: 50%;
            overflow: hidden;
            z-index: 1;
            transform-origin: top;
            backface-visibility: hidden;
            border-bottom-left-radius: 1.5rem; 
            border-bottom-right-radius: 1.5rem;
        }

        .card-content {
            position: absolute;
            left: 50%;
            transform: translateX(-50%);
            height: 200%;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .upper-card .card-content { top: 0; }
        .lower-card .card-content { bottom: 0; }

        @keyframes flipDown {
            0% { transform: rotateX(0deg); }
            100% { transform: rotateX(-180deg); }
        }

        .animate-flip-down {
            animation: flipDown 0.6s cubic-bezier(0.4, 0.0, 0.2, 1) forwards;
        }
        
        .split-line {
            position: absolute;
            top: 50%;
            left: 0;
            width: 100%;
            height: 2px;
            z-index: 20;
            transform: translateY(-50%);
        }
        
        .theme-transition {
            transition: background-color 0.5s ease, color 0.5s ease;
        }

        @media (max-width: 640px) {
            .upper-card, .lower-card {
                border-radius: 1rem;
            }
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect } = React;

        const themes = [
            {
                id: 'default',
                bg: 'bg-[#f3f4f6]',
                cardBg: 'bg-white',
                text: 'text-black',
                subText: 'text-gray-400',
                line: 'bg-gray-100'
            },
            {
                id: 'pink',
                bg: 'bg-[#ffebf3]',
                cardBg: 'bg-[#fabbd6]',
                text: 'text-white',
                subText: 'text-[#ffe4ef]',
                line: 'bg-white/10'
            },
            {
                id: 'blue',
                bg: 'bg-[#e0f2fe]',
                cardBg: 'bg-[#7dd3fc]',
                text: 'text-white',
                subText: 'text-[#e0f2fe]',
                line: 'bg-white/10'
            },
            {
                id: 'dark',
                bg: 'bg-[#09090b]',
                cardBg: 'bg-[#27272a]',
                text: 'text-[#f4f4f5]',
                subText: 'text-[#71717a]',
                line: 'bg-black/20'
            }
        ];

        const FlipUnit = ({ value, label, theme }) => {
            const [isFlipping, setIsFlipping] = useState(false);
            const [displayValue, setDisplayValue] = useState(value);
            const [oldValue, setOldValue] = useState(value);

            useEffect(() => {
                if (value !== displayValue) {
                    setOldValue(displayValue);
                    setDisplayValue(value);
                    setIsFlipping(true);
                    const timer = setTimeout(() => setIsFlipping(false), 600);
                    return () => clearTimeout(timer);
                }
            }, [value, displayValue]);
            
            const cardClass = `${theme.cardBg} ${theme.text} w-[42vw] h-[35vw] sm:w-64 sm:h-52 md:w-80 md:h-64 rounded-2xl sm:rounded-3xl shadow-xl relative flex flex-col justify-center items-center perspective-container theme-transition`;
            const textClass = `text-[25vw] sm:text-[10rem] md:text-[12rem] font-bold leading-none tracking-tighter select-none card-content`;

            return (
                <div className={cardClass}>
                    {label && (
                        <span className={`absolute top-3 left-4 sm:top-4 sm:left-6 text-base sm:text-2xl font-bold ${theme.subText} z-30 theme-transition`}>
                            {label}
                        </span>
                    )}

                    <div className="absolute inset-0 flex items-center justify-center rounded-2xl sm:rounded-3xl overflow-hidden">
                        <div className={`upper-card ${theme.cardBg}`}>
                             <span className={textClass}>{displayValue}</span>
                        </div>
                        <div className={`lower-card ${theme.cardBg}`}>
                             <span className={textClass}>{displayValue}</span>
                        </div>
                    </div>

                    {isFlipping && (
                        <div className={`upper-card ${theme.cardBg} z-50 origin-bottom animate-flip-down`}>
                            <span className={textClass}>{oldValue}</span>
                        </div>
                    )}
                     
                    <div className={`split-line ${theme.line}`}></div>
                </div>
            );
        };

        const App = () => {
            const [time, setTime] = useState(new Date());
            const [is24Hour, setIs24Hour] = useState(false);
            const [themeIndex, setThemeIndex] = useState(0);

            useEffect(() => {
                const timer = setInterval(() => setTime(new Date()), 1000);
                return () => clearInterval(timer);
            }, []);

            const currentTheme = themes[themeIndex];

            let hours = time.getHours();
            const minutes = time.getMinutes();
            const isPm = hours >= 12;
            
            let displayHours = hours;
            let amPmLabel = "";

            if (!is24Hour) {
                amPmLabel = isPm ? "PM" : "AM";
                displayHours = hours % 12 || 12;
            }

            const hourStr = displayHours.toString();
            const minStr = minutes.toString().padStart(2, '0');

            return (
                <div className={`w-full h-screen ${currentTheme.bg} flex items-center justify-center theme-transition relative select-none p-4`}>
                    
                    <div className="flex gap-4 sm:gap-8 items-center max-w-full">
                        <FlipUnit 
                            value={hourStr} 
                            label={!is24Hour ? amPmLabel : null} 
                            theme={currentTheme}
                        />
                        <FlipUnit 
                            value={minStr} 
                            theme={currentTheme}
                        />
                    </div>

                    <div className="fixed bottom-6 right-6 sm:bottom-10 sm:right-10 flex items-center gap-6">
                        <button 
                            onClick={() => setIs24Hour(!is24Hour)}
                            className={`text-xl sm:text-2xl font-bold cursor-pointer transition-opacity hover:opacity-70 ${currentTheme.text === 'text-black' ? 'text-black' : 'text-white'}`}
                        >
                            24h
                        </button>

                        <button 
                            onClick={() => setThemeIndex((themeIndex + 1) % themes.length)}
                            className={`p-1 rounded-full transition-transform hover:scale-110 active:scale-95 cursor-pointer ${currentTheme.text === 'text-black' ? 'text-black' : 'text-white'}`}
                        >
                            <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><circle cx="13.5" cy="6.5" r=".5"/><circle cx="17.5" cy="10.5" r=".5"/><circle cx="8.5" cy="7.5" r=".5"/><circle cx="6.5" cy="12.5" r=".5"/><path d="M12 2C6.5 2 2 6.5 2 12s4.5 10 10 10c.92 0 1.7-.72 1.55-1.63A4 4 0 0 0 17 16h1c2.2 0 4-1.8 4-4 0-3.31-4.48-6-10-6Z"/></svg>
                        </button>
                    </div>
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
