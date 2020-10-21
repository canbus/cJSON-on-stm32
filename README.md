# cJSON-on-stm32
//https://github.com/DaveGamble/cJSON

//create a object with a list 
//NOTE: Returns a heap allocated string, you are required to free it after use.
void *createJS(void)
{
    cJSON *root = cJSON_CreateObject(); /*create json string root*/
    if(!root) {
        DEBUG("get root faild !\n");
        goto EXIT;
    }else DEBUG("get root success!\n");

    {			
				cJSON *name = cJSON_CreateString("Jack");
			  cJSON_AddItemToObject(root,"name",name); //添加”name":"Jack"
				{
					cJSON *body = cJSON_CreateObject();
					cJSON_AddItemToObject(root,"body",body); //添加"body":{}
					
					cJSON_AddItemToObject(body,"height",cJSON_CreateNumber(175));
					cJSON_AddItemToObject(body,"weight",cJSON_CreateString("75kg"));
				}
				{
					cJSON *skills = cJSON_CreateArray();
					cJSON_AddItemToObject(root,"skill",skills);//添加"skill":[]
					{
						cJSON *skill = cJSON_CreateObject();
						cJSON_AddItemToArray(skills,skill);
						cJSON_AddItemToObject(skill,"lan",cJSON_CreateString("C"));
						cJSON_AddItemToObject(skill,"level",cJSON_CreateString("very good"));
					}
					{
						cJSON *skill = cJSON_CreateObject();
						cJSON_AddItemToArray(skills,skill);
						cJSON_AddItemToObject(skill,"lan",cJSON_CreateString("Java"));
						cJSON_AddItemToObject(skill,"level",cJSON_CreateString("good"));
					}
				}
				{ //输出
					char *s = cJSON_PrintUnformatted(root);//cJSON_Print(root); //带格式输出
					if(s){
							DEBUG("create js string is %s\n",s);
							free(s);
					}
				}
    }
		{
			char *ret = cJSON_PrintUnformatted(root);
			cJSON_Delete(root);
			return ret;
		}
EXIT:
    return 0;
}

int parseJS(char *jsonString)
{
	//"{"name":"Jack","body":{"height":175,"weight":"75kg"},"skill":[{"lan":"C","level":"very good"},{"lan":"Java","level":"good"}]}
	//1.先将普通的json串处理成json对象,注意：这个对像在解析完成后，需要释放掉
	cJSON *root = cJSON_Parse(jsonString); 
	
	//2.拿关键字，但如果关键字还有父层,就需要先从父层开拿，所谓剥洋葱是也
	cJSON *name = cJSON_GetObjectItem(root,"name");
	if(!name){
		DEBUG("no name\n");
		return -1;
	}
	if(name->type == cJSON_String)
		DEBUG("%s:%s\n",name->string,name->valuestring);
	
	{
		cJSON *body = cJSON_GetObjectItem(root,"body"); //解析内带的对像
		if(body->type == cJSON_Object){
			cJSON *height = cJSON_GetObjectItem(body,"height");
			if(height->type == cJSON_Number)
				DEBUG("%s:%d\n",height->string,height->valueint);
		}
	}
	{
		cJSON *skills = cJSON_GetObjectItem(root,"skill"); //解析数组
		if(skills->type == cJSON_Array){
			int i,size = cJSON_GetArraySize(skills);
			for(i=0;i<size;i++){
				cJSON *skill = cJSON_GetArrayItem(skills,i);
				cJSON *skill_lan = cJSON_GetObjectItem(skill,"lan");
				DEBUG("skill:%s:%s\n",skill_lan->string,skill_lan->valuestring);
			}
		}
	}
	
	if(!root)
		cJSON_Delete(root);
}
