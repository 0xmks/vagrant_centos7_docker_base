FROM jwilder/nginx-proxy:v0.9.0

# 転送サイズ制限設定の調整等
RUN { \
      echo 'server_tokens off;'; \
      echo 'client_max_body_size 100g;'; \
    } > /etc/nginx/conf.d/my_proxy.conf

