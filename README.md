# Introducere
Aplicația dezvoltată are ca scop facilizarea transmiterii de mail-uri de către un utilizator în mai multe limbi internaționale și, la cerere, analiza fotografiei de profil a acestuia. Aceasta este construită folosind Node.js și React.js, fiind o aplicație de tip REST. 
# Descrierea problemei
Aplicația se adresează oricărei persoane care folosește un cont Google și dorește să transmită mail-uri prietenilor săi din străinătate. Autentificarea în aplicație se realizează facil, direct prin intermediul unui cont de Gmail. După autentificare numele și fotografia de profil a utilizatorului vor apărea în interfața principala, semn că aceasta s-a realizat cu success. Funcționalitatea principala este cea de traducere a unui mesaj într-una sau mai multe limbi internaționale și transmiterea acestuia către un prieten. De asemenea, utilizatorul va putea urmări și mesajele care au fost trimise anterior. Nu în ultimul rând, aplicația poate transmite utilizatorului un mesaj la cerere bazat pe analiza fotografiei de profil a acestuia.
# Descriere API
	Google Authentication API
Pentru componenta de autentificare a utilizatorului pentru a folosi aplicația și a accesa date despre utilizator, am utilizat API-ul de autentificare prin Google, prin intermediul Firebase Authentication. În acest mod, credentialele utilizatorului sunt transmise automat către Firebase Authentication SDK unde vor fi verificate pentru a returna un răspuns.
	Google Translation API
Pentru traducerea mesajelor în diverse limbi selectate se utilizează Translation API care folosește tehnologia de traducere automată neuronală de la Google pentru a traduce instantaneu texte în mai mult de o sută de limbi.
	Google Cloud Vision API
Acesta este un API care înțelege imaginile cu o precizie de vârf în industrie și poate clasifică imaginile după etichetele personalizate folosind AutoML Vision, deoarece detectează obiectele și fetele. API-ul este apelat prin intermediul axios și are rolul de a analiza imaginea de profil a utilizatorului aplicației pentru a prelua diverse elemente.
# Fluxul de date
Pentru autentificarea utilizatorului, Firebase vine la pachet cu metode ajutătoare ce permit autentificarea cu servicii precum Google. Apelul către serviciul cloud Firebase Authentication este următorul:

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app); 

const provider = new GoogleAuthProvider();
export const signInWithGoogle = () => {
    signInWithPopup(auth, provider)
    .then((result) => {
       const name = result.user.displayName;
       const email = result.user.email;
       const profilePic = result.user.photoURL;

       localStorage.setItem("name", name);
       localStorage.setItem("email", email);
       localStorage.setItem("profilePic", profilePic);
    })
    .catch((error) => {
        console.log(error);
    });
}

Astfel se vor prelua numele, email-ul și poza de profil a utilizatorului, care vor fi stocate local și disponibile întregii aplicații.
După autentificare, utilizatorul va fi întâmpinat de fereastra principală de unde poate trimite mesaje. Este necesară completarea câmpurilor cu adresa de mail a receptorului și cu mesajul dorit pentru ca utilizatorul să poată selecta limba în care mesajul să fie transmis și pentru a realiza request-ul. Câmpul cu numele expeditorului este deja completat în urma autentificarii.
Textul va fi preluat din input și cedat către parametrii necesari. Se vor realiza două request-uri: cel de GET pe metodă `/utils/translate` și cel de POST pe metoda  `/messages`. Obiectele primite de la API vor fi mapate pe componenta MessageSubmit.
În pagină principala, utilizatorul poate observa de asemenea toate mesajele care au fost trimise, realizându-se o metodă de GET pe `/messages`:

router.get("/", (req, res) => {
    connection.query("SELECT * FROM messages", (err, results) => {
        if (err) {
            console.log(err);
            return res.send(err);
        }

        return res.json({
            messages: results,
        });
    });
});

În final, la cererea utilizatorului prin apăsarea unui buton, imaginea de profil a utilizatorului va putea fi analizată prin Cloud Vision API, care va returna un mesaj cu elementele principale din fotografie în pagina principală. Query parameter necesar pentru request-ul de GET este reprezentat de link-ul fotografiei contului de Google asociat utilizatorului.
 const handlePicture= async (e) => {
        const pictureLink = localStorage.getItem("profilePic");

        try {
            let responseImage = await axios.get(`${process.env.REACT_APP_API_URL}/utils/labels`,{params:{link: pictureLink}});

            console.log(responseImage.data.dominantColors);
                if(responseImage.status === 200) {
                    alert(`In your profile picture i can distict beautiful elements like a ${responseImage.data.labels}. \nI like your photo and i beleive you have a great personality! `);
                }
        }
        catch (error) {
            alert('Can not see your picture');
            console.log(error);
        }
    }

# Capturi de ecran
Autentificarea prin API
![image](https://user-images.githubusercontent.com/72612275/168168438-a89f4bfe-ee15-4331-97d9-d64875d6d17f.png)

Fereastra principală de transmitere mail-uri
![image](https://user-images.githubusercontent.com/72612275/168168452-24761284-4183-41fc-bfa4-5613464ed138.png)

Analiza imaginii de profil a utilizatorului
![image](https://user-images.githubusercontent.com/72612275/168168468-676bf34c-0622-4404-abc5-74347ec85464.png)


# Referințe
•	Documentație Seminar Cloud Computing
•	https://firebase.google.com/docs/auth
•	https://cloud.google.com/translate
•	https://cloud.google.com/vision
•	https://restfulapi.net/
•	
•	https://tailwindcss.com/

# Publicarea aplicației
Aplicația este publicată pe platforma Heroku și poate fi accesată la adresa: 
