# Réalisé par G. Chollet


```python
import streamlit as st
import requests
st.set_page_config(page_title="Gen Z", page_icon="/Users/Laetitia/Documents/Wild/Projet_2/KPI/logo_genz.png")
# Mettre à jour les URLs pour correspondre à votre API
API_URL = "http://192.168.1.27:8503"
#RECOMMEND_MOVIE_API_URL = "http://127.0.0.1:8000/recommend_movies"
# Votre style CSS existant
st.markdown(
    """
    <style>
    .stApp {
        background-image: url('/Users/Laetitia/Documents/Wild/Projet_2/KPI/fond_dashboard_camera.png');
        background-size: cover;
        background-repeat: no-repeat;
        background-position: center;
    }
    .stTextInput > div > div > input {
        background-color: #2C2C2C;
        color: white;
        font-size: 1.5em;
    }
    .stButton > button {
        background-color: #FF5757;
        color: white;
    }
    .stMarkdown p {
        color: white;
    }
    h1 {
        color: #FF5757;
    }
    .large-text {
        font-size: 1.5em;
        font-weight: bold;
        color: #FF5757;
    }
    .large-number {
        font-size: 1.5em;
        color: white;
        font-weight: bold;
    }
    hr {
        border: 1px solid #FF5757;
    }
    </style>
    """,
    unsafe_allow_html=True
)
# Interface utilisateur
st.image("/Users/Laetitia/Documents/Wild/Projet_2/KPI/logo_genz.png", width=300)
st.title("A Very Creuse Cinema", anchor="start")
st.write("")
st.write("")
st.write("")
st.write("")
# Définition de la fonction pour charger les images
def load_image(image_path):
    base_url = "https://image.tmdb.org/t/p/w200"
    if not image_path:
        default_image = "/Users/Laetitia/Documents/Wild/Projet_2/1000_F_296273594_AH1UfwzyQLwQr9A4JDOtNeDepzNNIpXJ.jpg"
        st.image(default_image, width=300)
    elif image_path.startswith("http"):
        st.image(image_path, width=300)
    else:
        full_url = f"{base_url}{image_path}"
        st.image(full_url, width=300)
# Fonction pour limiter ou afficher le texte complet avec un bouton
def afficher_synopsis_complet(synopsis, index):
    key_expand = f"expand_synopsis_{index}"
    if st.session_state.get(key_expand, False):
        # Afficher le synopsis complet
        st.write(f"{synopsis}")
        if st.button("Réduire", key=f"reduce_{key_expand}"):
            st.session_state[key_expand] = False
    else:
        # Afficher le synopsis limité
        texte_limite = limiter_synopsis(synopsis, max_caracteres=500)
        st.write(texte_limite)
        if len(synopsis) > 500:
            if st.button("Lire la suite", key=f"expand_{key_expand}"):
                st.session_state[key_expand] = True
# Champ de recherche
movie_name = st.text_input(
    label="",
    placeholder="Entrez le nom d'un film",
    key="movie_input"
)
# Fonction pour limiter le texte à 500 caractères
def limiter_synopsis(text, max_caracteres=500):
    if text is None:
        return ""  # Si le texte est None, retournez une chaîne vide
    if isinstance(text, str) and len(text) > max_caracteres:
        texte_limite = text[:max_caracteres] + "..."  # Ajouter une ellipse si le texte dépasse les 500 caractères
    else:
        texte_limite = text  # Sinon, retournez le texte tel quel
    return texte_limite
# Logique de recherche et recommandation
if movie_name:
    try:
        # Recherche de films
        response = requests.get(API_URL, params={"morceau_titre": movie_name})  # Changé de title à morceau_titre
        if response.status_code == 200:
            movies = response.json().get("movies", [])
            if movies:
                # Liste déroulante avec les films trouvés
                movies_with_placeholder = ["Sélectionnez un film"] + movies  # Simplifié car l'API renvoie directement les titres
                selected_movie = st.selectbox(
                    "Veuillez sélectionner un film dans la liste",
                    movies_with_placeholder,
                    key="movie_selectbox"
                )
                if selected_movie != "Sélectionnez un film":
                    if not st.session_state.get("show_recommendations", False):
                        st.session_state.show_recommendations = True
                    if st.session_state.show_recommendations:
                        st.title("VOS PROCHAINS COUPS DE COEUR")
                        # Obtenir les recommandations
                        recommend_response = requests.get(
                            API_URL,
                            params={"title": selected_movie}
                        )
                        if recommend_response.status_code == 200:
                            recommended_movies = recommend_response.json().get("recommended_movies", [])
                            if recommended_movies:
                                for index, movie in enumerate(recommended_movies, start=1):
                                    title = movie['title']  # Utilise directement title
                                    note = movie['note']
                                    runtime = int(movie['runtimeMinutes'])  # Suppression des décimales
                                    synopsis = movie['synopsis']  # Pas de limitation ici
                                    date_sortie = movie['date_sortie']
                                    affiche = movie['affiche']  # Changé de poster_path
                                    # Affichage en colonnes
                                    col1, col2 = st.columns([1, 2])
                                    with col1:
                                        load_image(affiche)
                                    with col2:
                                        st.write(f"<span class='large-number'>#{index}</span> &nbsp; <span class='large-text'>{title}</span>", unsafe_allow_html=True)
                                        st.write(f"<b><u>Synopsis :</u></b>", unsafe_allow_html=True)
                                        afficher_synopsis_complet(synopsis, index)  # Appeler la fonction pour gérer le synopsis
                                        st.write("")
                                        st.write("")
                                        st.write("")
                                        st.write(f"<b>Note IMDb :</u></b> {note}", unsafe_allow_html=True)
                                        st.write(f"<b>Durée :</u></b> {runtime} minutes", unsafe_allow_html=True)
                                        st.write(f"<b>Année de sortie :</u></b> {date_sortie}", unsafe_allow_html=True)
                                    st.markdown("<hr>", unsafe_allow_html=True)  # Ligne de séparation rouge
                            else:
                                st.warning(f"Aucune recommandation trouvée pour '{selected_movie}'.")
                        else:
                            st.error(f"Erreur de recommandation : {recommend_response.json().get('detail')}")
            else:
                st.warning(f"Aucun film trouvé contenant '{movie_name}'.")
        else:
            st.error("Erreur lors de la recherche de films")
    except Exception as e:
        st.error(f"Une erreur s'est produite : {e}")
```
