```
(() => {
    const normalize = str => str.replace(/\s+/g, ' ').trim();

    const MARK = '.';
    let devMode = false;

    const parsed = JSON.parse(decodeURIComponent(escape(atob(data))));

    const answers = parsed.d.sl.g.reduce((first, theme) => {
        const third = theme.S.reduce((second, question) => {
            try {
                const right = question.C.chs.filter(option => option.c);
                const description = normalize(question.D.d[0].split('\n')[0]);
                const rightTexts = right.map(option => normalize(option.t.d[0]));

                if (second[description]) {
                    second[description] = second[description].concat(rightTexts);
                } else {
                    second[description] = rightTexts;
                }
            } catch (_) {}

            return second;
        }, {});

        return { ...first, ...third };
    }, {});

    console.log('answers:', answers);

    const getAllSpans = () => {
        return Array.from(document.querySelectorAll('.player-shape-view span'));
    };

    const getOriginalText = el => {
        return el.dataset.devOriginalText || el.textContent;
    };

    const markAnswer = el => {
        if (el.dataset.devMarked === '1') return;

        el.dataset.devMarked = '1';
        el.dataset.devOriginalText = el.textContent;
        el.innerText = el.textContent + MARK;
    };

    const unmarkAnswer = el => {
        if (el.dataset.devMarked !== '1') return;

        el.innerText = el.dataset.devOriginalText || el.textContent.replace(/\.$/, '');

        delete el.dataset.devMarked;
        delete el.dataset.devOriginalText;
    };

    const unmarkAll = () => {
        document
            .querySelectorAll('[data-dev-marked="1"]')
            .forEach(unmarkAnswer);
    };

    const updateDevHints = () => {
        try {
            const allSpans = getAllSpans();

            if (!devMode) {
                unmarkAll();
                return;
            }

            const currentQuestionEl = allSpans.find(el => {
                const text = normalize(getOriginalText(el));
                return Object.prototype.hasOwnProperty.call(answers, text);
            });

            if (!currentQuestionEl) {
                unmarkAll();
                return;
            }

            const questionText = normalize(getOriginalText(currentQuestionEl));
            const rightAnswers = answers[questionText];

            if (!rightAnswers) return;

            const rightAnswerSet = new Set(rightAnswers.map(normalize));
            const currentRightElements = new Set();

            allSpans.forEach(el => {
                const text = normalize(getOriginalText(el));

                if (rightAnswerSet.has(text)) {
                    markAnswer(el);
                    currentRightElements.add(el);
                }
            });

            document
                .querySelectorAll('[data-dev-marked="1"]')
                .forEach(el => {
                    if (!currentRightElements.has(el)) {
                        unmarkAnswer(el);
                    }
                });
        } catch (error) {
            console.warn('dev mode error:', error);
        }
    };

    document.addEventListener('keydown', event => {
        const isTyping =
            event.target.tagName === 'INPUT' ||
            event.target.tagName === 'TEXTAREA' ||
            event.target.isContentEditable;

        if (isTyping) return;

        if (event.repeat) return;

        if (event.code === 'Digit1' || event.code === 'Numpad1') {
            devMode = !devMode;

            console.log('devMode:', devMode ? 'ON' : 'OFF');

            updateDevHints();
        }
    });

    setInterval(updateDevHints, 500);
})();


```
