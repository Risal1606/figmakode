Kort refleksion
I denne opgave har fokus været at bygge en løsning med moderne CSS, men stadig på en måde hvor siden fungerer, selv hvis ikke alle features er understøttet i browseren. En af de største udfordringer var at få de nye CSS-teknikker til at spille sammen med et layout, der stadig skulle være stabilt og overskueligt. Især anchor positioning krævede lidt ekstra arbejde, fordi det ikke bare kunne laves direkte uden at tænke på fallback.

Et sted hvor løsningen fungerede godt, var brugen af custom properties og mere moderne CSS-funktioner. For eksempel blev der arbejdet med tokens i :root, så farver, spacing og størrelser kunne genbruges flere steder i projektet. Det gjorde CSS’en nemmere at vedligeholde, og det gav også en mere ensartet styling på tværs af komponenter.

Et konkret eksempel er login-panelet, hvor der
 både er lavet en almindelig fallback med position: absolute, og derefter en forbedret løsning med anchor positioning:

 .login-panel {
  position: absolute;
  top: calc(100% + 1rem);
  right: 0;
}

@supports (anchor-name: --test) and (position-anchor: --test) {
  .site-header__login {
    anchor-name: --login-trigger;
  }

  .login-panel {
    top: auto;
    right: auto;
    position-anchor: --login-trigger;
    inset-block-start: calc(anchor(bottom) + 1rem);
    inset-inline-end: anchor(right);
  }
}

Det er brugbart, fordi løsningen først virker på en simpel måde i alle browsere, og derefter bliver forbedret i browsere, der understøtter de nye features. Det er også et eksempel på progressive enhancement, fordi basisoplevelsen stadig virker uden den nyeste CSS.

Defensive CSS er især tænkt ind i forhold til layout og responsive løsninger. Der er blandt andet brugt width: min(23rem, calc(100vw - 2rem));, så login-panelet ikke bliver for bredt på små skærme. Derudover er der lavet media queries, så elementer flytter sig og centreres anderledes på smallere skærme. Det gør løsningen mere robust og mindsker risikoen for overflow eller mærkelige placeringer.

Der er også brugt progressive enhancement andre steder i løsningen, fx hvor nyere features kun bliver aktiveret med @supports. På den måde bliver browseren ikke "straffet", hvis den ikke understøtter det nyeste, men får bare en simplere version. Det passer godt til tanken fra undervisningen om, at en hjemmeside først og fremmest skal fungere, og derefter kan forbedres.

CSS’en er organiseret sådan, at de globale regler ligger øverst, fx reset, variabler, generelle farver, typography og layoutregler, som bruges flere steder. Derefter kommer komponent-specifik CSS, fx til header, login-panel, FAQ og andre sektioner. Det gjorde det lettere at finde rundt i filen, fordi de overordnede regler ikke blev blandet sammen med detaljerne for de enkelte komponenter.

Alt i alt var opgaven god til at vise forskellen på bare at få noget til at se rigtigt ud, og faktisk at bygge det på en måde, der er gennemtænkt. Det mest vellykkede i løsningen er, at den kombinerer moderne CSS med fallback og responsive hensyn, så den både virker praktisk og viser de teknikker, der har været arbejdet med i undervisningen.
