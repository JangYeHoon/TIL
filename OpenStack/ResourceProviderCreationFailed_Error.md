## 원인

오픈스택에서 VM 생성 시 특정 호스트 서버에서 생성이 안되는 문제가 발생하여 nova-compute 로그를 확인해보니 아래 에러 로그가 있었습니다.

- **ResourceProviderCreationFailed:Failed to create resource provider**

해당 오류는 vm migration이나 evacuate 진행 시에 실패하여 오류로 있는 VM을 따로 처리하지 않고 삭제하여 발생하는 오류입니다.



## 해결 방법

오류에 대한 해결 방법을 버그 리포터에서 찾았는데 만약 해당 VM을 복원할 필요가 있는 경우 상당히 복잡합니다.

해당 방법은 해보지는 않았지만 일단 간략히 정리하고 자세한 과정은 https://bugzilla.redhat.com/show_bug.cgi?id=1774243#c2 에서 확인하시면 됩니다.

### 방법 1

- VM 복원이 필요하지 않을 때 사용하는 방법입니다.

```bash
# resource provider -> rp
$ openstack resource provider list --name $host_name
$ openstack resource provider show --allocations $rp_uuid # 이전 명령어에서 나온 uuid
$ openstack resource provider allocation delete $server_uuid  # 해당 rp에 할당된 서버 삭제
$ openstack resource provider delete $rp_uuid
$ systemctl restart nova-compute
```

### 방법 2

- VM 복원이 필요한 경우 사용하는 방법 입니다.

```bash
1. Get the resource provider UUID using the compute host name:
$ rp_uuid=$(openstack resource provider list --name compute-0 -f value -c uuid)

2. View the allocations for the compute host. This will show you all of the servers allocations:
$ openstack resource provider show --allocations $rp_uuid

3. Get a list of all servers running on the host:
$ openstack server list --host compute-0

4. For each server in the list:

4a. Get and **SAVE** the allocations for the server. You will need to restore them later:
$ openstack --os-placement-api-version 1.12 resource provider allocation show $server_uuid

4b. Delete the allocation for the server for compute-0:

If there was only one resource provider in the allocations from step 4a, delete all allocations on all compute hosts for the server:
$ openstack resource provider allocation delete $server_uuid

If there were multiple allocations in step 4a, delete only the allocation for compute-0 for the server by omitting $rp_uuid:
$ openstack --os-placement-api-version 1.12 resource provider allocation set $server_uuid \
    --project-id <project uuid from step 4a> \
    --user-id <user uuid from step 4a> \
    --allocation rp=<other rp uuid than compute host>,<resource class from step 4a>=<value from step 4a> \
    repeat --allocation for all the rest of the allocations for resource providers other than compute host

5. Once you have deleted and **SAVED** all the allocations from the servers on compute-0, delete the resource provider:
$ openstack resource provider delete $rp_uuid

6. Restart nova-compute. The resource providers should be re-created successfully.
$ systemctl restart nova-compute

7. Get the (new) resource provider UUID using the compute host name:
$ new_rp_uuid=$(openstack resource provider list --name compute-0 -f value -c uuid)

8. Restore all of the allocations for the servers on compute-0:

8a. Create (restore) the same allocations for the server on compute-0 that you **SAVED** from step 4a:
$ openstack --os-placement-api-version 1.12 resource provider allocation set $server_uuid \
    --project-id <project uuid from step 4a> \
    --user-id <user uuid from step 4a> \
    --allocation rp=$new_rp_uuid,<resource class from step 4a>=<value from step 4a> \
    repeat --allocation for all allocations you saved from step 4a
    
9. The compute host should now be recovered.
```

