FROM nginx

# SUpprimer le fichier de configuration par défaut
RUN rm /etc/nginx/conf.d/default.conf

# Copier le fichier de configuration personnalisé
COPY nginx.conf /etc/nginx/conf.d/default.conf
