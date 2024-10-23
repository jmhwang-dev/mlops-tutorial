# init:
# 	python3 -m venv .env
# 	. .env/bin/activate

install:
	pip install --upgrade pip && \
	pip install -r requirements.txt

lint:
	pylint --disable=R,C hello.py

test:
	pytest test_hello.py