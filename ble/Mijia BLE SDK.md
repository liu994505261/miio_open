# Mijia BLE SDK

## ���
Mijia BLE SDK��������оƬԭ��SDK�����ϣ�����С�� Profile���װ�õ���Ŀ���Ƿ�����������ж��ο������������¹��ܣ�
- ֧��С�������㲥Э�飺mi beacon
- ֧�� Mi Service���������ܼ�ͥAPP���豸��İ�ȫ���Ӽ���֤
- ʾ�����̣�Mi Demo����¼�������������С�����ܼ�ͥAPPֱ�ӿ�Ч������Ӧ��Ʒ���ƣ�С������������


## ֧��ƽ̨��
- Dialog - DA14585
- Cypress - CYW20706
- Nordic - nRF51822
- Marvell - 88w8777


## OTA
���ڹ̼�������Mijia BLE SDK��û���ṩͳһ��ʵ�֣�ֻ�ṩ�豸�İ汾�Ź淶�����ܼ�ͥAPP����ݰ汾���ж��Ƿ����¹̼���Ҫ����������������߼��ɲ�Ʒ�Լ�ʵ�֡�
 - �汾��ʾ����1.0.1_16,  sdk�汾��_Ӧ�ð汾�ţ�Ӧ�ð汾�Ų�Ҫ����4���ַ�
 - OTA����˵��
	 1. ���ܼ�ͥAPP���豸���Ӻ���Զ���ȡ�豸��ǰ�汾��
	 2. ���ƶ����°汾���бȽϣ������°汾��֪ͨ ���
	 3. �ɲ��������ʲôʱ����ʾ�û���������

	 
## ��ȫ����/��֤
Mi Serivce���������������Ͻ���һ����ȫ��֤�����̣����ڷ�ֹ�м��˹����Լ��طŹ�������һ�����Ѿ�������SDK�У��豸һ����APP�������������Ӻ󣬻��Զ���ʼ��ȫ��֤����ȫ��֤�Ľ����ص���Ӧ�ò�,���鿪�������յ�auth_fail����Ϣ���������Ͽ��������ӡ�

```C
int miss_auth_fail_ind_handler(ke_msg_id_t const msgid,
                                    struct miss_authFail_ind const *param,
                                    ke_task_id_t const dest_id,
                                    ke_task_id_t const src_id)
{
    /* TODO: Add application layer handler here */
    app_disconnect();
    LOG_DEBUG("app_task auth fail\r\n");
    return (KE_MSG_CONSUMED);
}

int miss_bond_succ_ind_handler(ke_msg_id_t const msgid,
                                    struct miss_bondSucc_ind const *param,
                                    ke_task_id_t const dest_id,
                                    ke_task_id_t const src_id)
{
    /* TODO: Add application layer handler here */
    LOG_DEBUG("app_task bond succ\r\n");
    return (KE_MSG_CONSUMED);
}
```


## ������
��Щ��Ʒ��Ҫ���ֻ�ά�ֳ����ӣ�MIUI�Ѿ���Mi Profile����֧�֣������ڲ���н���Ҫ�����ӵĲ�Ʒ���õ�ϵͳ�С�
