BACKUP_DIR = ~/backup

RUBY_SERVICE_NAME = isuda.ruby isutar.ruby
RUBY_SOURCE_DIR = ~/webapp/ruby

NGINX_ACCESS_LOG = /var/log/nginx/access.log
NGINX_ERROR_LOG = /var/log/nginx/error.log

MYSQL_USERNAME = isucon
MYSQL_LOG = /var/log/mysql/mysql.log
MYSQL_ERROR_LOG = /var/log/mysql/error.log
MYSQL_SLOW_LOG = /var/log/mysql/slow.log


.DEFAULT_GOAL := help

restart: ## Restart Services of Nginx, MySQL, and Ruby
	@make -s nginx-restart db-restart ruby-restart

ruby-log: ## Show Log Ruby Service
	@sudo journalctl -u ${arg} ${RUBY_SERVICE_NAME}

ruby-restart: ## Restart Ruby Service
	@echo -n "Restarting ${RUBY_SERVICE_NAME}... "
	@(sudo systemctl daemon-reload \
			&& cd ${RUBY_SOURCE_DIR} \
			&& bundle 1> /dev/null \
			&& sudo systemctl restart ${RUBY_SERVICE_NAME}) \
		&& echo 'done!'

nginx-restart: ## Restart Nginx
	@echo -n 'Restarting Nginx... '
	@sudo systemctl restart nginx \
		&& echo 'done!'

nginx-log: ## tail Nginx access.log [arg=<tail option>]
	@sudo tail ${arg} ${NGINX_ACCESS_LOG}

nginx-error-log: ## tail Nginx error.log [arg=<tail option>]
	@sudo tail ${arg} ${NGINX_ERROR_LOG}

nginx-backup: ## Make nginx.conf Backup to ~/backup/
	@echo -n "Saving nginx.conf to ${BACKUP_DIR}/nginx_`date "+%Y%m%d_%H%M%S".conf` ... "
	@cp /etc/nginx/nginx.conf ${BACKUP_DIR}/nginx_`date "+%Y%m%d_%H%M%S".conf` \
		&& echo 'done!'

alp: ## Run alp [arg=<alp option>]
	@sudo alp ${arg} ${NGINX_ACCESS_LOG}

db-restart: ## Restart MySQL
	@echo -n 'Restarting MySQL... '
	@sudo systemctl restart mysql \
		&& echo 'done!'

db-log: ## tail mysql.log [arg=<tail option>]
	@sudo tail ${arg} ${MYSQL_LOG}

db-error-log: ## tail error.log [arg=<tail option>]
	@sudo tail ${arg} ${MYSQL_ERROR_LOG}

db-slowlog: ## pt-query-digest slow.log
	@sudo pt-query-digest ${MYSQL_SLOW_LOG}

db-backup: ## Make MySQL Backup to ~/backup/
	@echo -n "Saving mysql/my.cnf to ${BACKUP_DIR}/my_`date "+%Y%m%d_%H%M%S".cnf` ... "
	@cp /etc/mysql/my.cnf ${BACKUP_DIR}/my_`date "+%Y%m%d_%H%M%S".cnf` \
		&& echo 'done!'
	@echo "Saving mysqldump to ${BACKUP_DIR}/mysqldump_`date "+%Y%m%d_%H%M%S"`.gz ... "
	@(mysqldump --single-transaction -u ${MYSQL_USERNAME} -p --all-databases \
		| gzip > ${BACKUP_DIR}/mysqldump_`date "+%Y%m%d_%H%M%S"`.gz) \
		&& echo 'done!'

db-restore: ## Restore MySQL from Backup [arg=<backup file full path>]
	@if [ -n "${arg}" ]; then \
			echo "Restoring from ${arg} ..."; \
			zcat -f ${arg} | mysql -u ${MYSQL_USERNAME} -p \
				&& echo 'done!'; \
		else \
			echo 'Please set arg=<backup file full path>'; \
		fi

service-status: ## Show Status of Service
	@sudo systemctl list-unit-files -t service

backup-all: ## Make Nginx and Mysql Bakup
	@make -s nginx-backup db-backup

clean-log:
	@sudo rm -f ${NGINX_ACCESS_LOG} ${NGINX_ERROR_LOG} ${MYSQL_LOG} ${MYSQL_SLOW_LOG} \
		&& echo "${NGINX_ACCESS_LOG}, ${NGINX_ERROR_LOG}, ${MYSQL_LOG}, and ${MYSQL_SLOW_LOG} have been removed!"

.PHONY: help

help:
	@echo 'Usage: make <command> [arg=<arguments>]'
	@grep -E '^[a-z0-9A-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'


