# Implementovaná funkcionalita
Program sa dokáže pripojiť k serveru cez TCP. Program dokáže posielať a príjmať správy ako cez protokol TCP tak UDP. Program asynchronne detekuje signál prerušenia a správne ukončí spojenie. V móde UDP k správe od uživatela pridá hlavičku, a v odpovedi servera hlavičku oddelí predtým ako správu vypíše. Program kontroluje argumenty a pri nesprávne zadaných parametroch, alebo pri parametre --help vypíše nápovedu.

# Známe problémy
Pri testovaní na referenčnom stroji, na systéme Ubuntu a pri porovnávaní s programom netcat neboli nájdené žiadne limitácie.