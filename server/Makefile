include Makefile-flash-att
include Makefile-vllm

unit-tests:
	pytest -s -vv -m "not private" tests

gen-server:
	# Compile protos
	pip3.9 install grpcio-tools==1.51.1 mypy-protobuf==3.4.0 'types-protobuf>=3.20.4' --no-cache-dir
	mkdir text_generation_server/pb || true
	python3.9 -m grpc_tools.protoc -I../proto --python_out=text_generation_server/pb \
		--grpc_python_out=text_generation_server/pb --mypy_out=text_generation_server/pb ../proto/generate.proto
	find text_generation_server/pb/ -type f -name "*.py" -print0 -exec sed -i -e 's/^\(import.*pb2\)/from . \1/g' {} \;
	touch text_generation_server/pb/__init__.py

install-torch:
	# Install specific version of torch
	pip3.9 install torch --extra-index-url https://download.pytorch.org/whl/cu118 --no-cache-dir

install: gen-server install-torch
	pip3.9 install pip --upgrade
	pip3.9 install -r requirements.txt
	pip3.9 install -e ".[bnb, accelerate]"

run-dev:
	SAFETENSORS_FAST_GPU=1 python3.9 -m torch.distributed.run --nproc_per_node=2 text_generation_server/cli.py serve bigscience/bloom-560m --sharded

export-requirements:
	poetry export -o requirements.txt -E bnb --without-hashes
