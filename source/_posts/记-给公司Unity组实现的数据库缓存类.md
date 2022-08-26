title: 记_给公司Unity小组实现的基于Sqlite缓存Cache类
date: 2019-04-18 10:28:51
tags:
- Unity
- C#
- Sqlite
categories: Unity
---

### 需求
1. 所有线上接口管理至本地缓存
2. 使用面向对象
3. db使用sqlite

### 设计思路

线上拉取数据,缓存至本地,读取本地版本,对比线上版本号,如有更新,更新本地数据库数据,如没有,则读取本地.

### 核心代码

> `IDbDao.cs` 接口定义

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace Assets.Scripts.DbDao
{
    public interface IDbDao<T> where T : class, new()
    {
        void setCache(string r);
    }
}

```

> `CacheOption.cs` 实现


```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Assets.Scripts.Repository;
using Assets.Scripts.Model;
using Assets.Scripts.Infrastructure;
using UnityEngine;
using Newtonsoft.Json;

namespace Assets.Scripts.DbDao
{
    public class CacheOption<T> : IDbDao<T> where T : class, new()
    {
        public void setCache_dbPath(string r, string path, bool is_append = false)
        {
            var repository = new RemoteRepository<GetStruct>();
            repository.Get<Response<T>>(r, (response) =>
            {
                Debug.Log(JsonConvert.SerializeObject(response));
                if (response.List != null && response.List.Count != 0)
                {
                   
                    var db = new DbRepository<T>();
                    db.CreateDb(path);
                    if (!is_append)
                    {
                        db.DropTable();
                    }
                    db.CreateTable();
                    db.InsertAll(response.List);//更新远程数据源
                    db.Close();
                }
                else
                {
                    Debug.Log("Struct List data is null ");
                }
            });
        }
        //实现写入缓存共有类
        public void setCache(string r)
        {
            var repository = new RemoteRepository<GetStruct>();

            var version = new DbRepository<TableVersions>();
            version.DataService("vesali.db");
            version.CreateTable();
            TableVersions tv = version.SelectOne<TableVersions>((tmpT) =>
            {
                if (tmpT.table_name == typeof(T).Name)
                {
                    return true;
                }
                return false;

            });
            var struct_version = "-1";
            if (tv != null)
            {
                struct_version = tv.version;
            }
            
            
            repository.Get<Response<T>>(r, new GetStruct { Version = struct_version, device = SystemInfo.deviceUniqueIdentifier, os = Enum.GetName(typeof(asset_platform), PublicClass.platform) , level = ((int)PublicClass.Quality).ToString(), softVersion = PublicClass.get_version() }, (response) =>
                       {
                           if (response.List != null && response.List.Count != 0)
                           {
                               Debug.Log("============读取数据库缓存更新，从远程拉取数据，更新版本号=================== ");
                               //如果有数据,更新数据和版本
                               if (tv == null)
                               {
                                   version.Insert(new TableVersions { table_name = typeof(T).Name, version = response.maxVersion });
                               }
                               else
                               {
                                   version.Update(new TableVersions { table_name = typeof(T).Name, version = response.maxVersion });
                               }
                               version.Close();
                               var db = new DbRepository<T>();
                               db.DataService("vesali.db");
                               db.DropTable();
                               db.CreateTable();
                               db.InsertAll(response.List);//更新远程数据源
                               db.Close();
                           }
                           else
                           {
                               Debug.Log("Struct List data is null ");
                               // Debug.Log("============读取数据库缓存=================== ");
                               //读取数据库缓存
                           }
                           PublicClass.data_list_count++;
                       });
            //throw new NotImplementedException();
        }
    }
}

```

> `IRepository.cs` 对基本数据操作接口定义,如增删改查


```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace Assets.Scripts.Repository
{
    public interface IRepository<T> where T:class,new()
    {
        void Insert(T instance);
        void Delete(T instance);
        void Update(T instance);
        IEnumerable<T> Select(Func<T,bool> func );
    }
}

```


>  `DbRepository.cs`数据库脚本实现

```
using System;
using System.IO;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using SQLite4Unity3d;
using UnityEngine;

namespace Assets.Scripts.Repository
{
    public class DbRepository<T> : IRepository<T> where T : class, new()
    {
        //public delegate void DoConnection(String str);
        private SQLiteConnection _connection;
        //private string v;

        //public DbRepository(string v)
        //{
        //    this.v = v;
        //}
        public void CreateTable()
        {
            _connection.CreateTable<T>();

        }
        public TableQuery<T> getTableQuerry()
        {
            return _connection.Table<T>();
        }
        public void DropTable()
        {
            _connection.DropTable<T>();
        }

        public void Delete(T instance)
        {
            try
            {
                _connection.Delete(instance);
            }
            catch (Exception e)
            {
                throw new Exception(e.ToString());
            }
        }

        public void Insert(T instance)
        {
            try
            {
                _connection.Insert(instance);
            }
            catch (Exception e)
            {
                throw new Exception(e.ToString());
            }
        }

        public void InsertAll(T[] instance)
        {
            try
            {
                _connection.InsertAll(instance);
            }
            catch (Exception e)
            {
                throw new Exception(e.ToString());
            }
        }


        public void InsertAll(List<T> instance)
        {
            try
            {
                _connection.InsertAll(instance);
            }
            catch (Exception e)
            {
                throw new Exception(e.ToString());
            }
        }

        public T SelectOne<R>(Func<T, bool> func) where R : class, new()
        {
            return _connection.Table<T>().Where(func).FirstOrDefault();
        }

        public IEnumerable<T> Select<R>(Func<T, bool> func) where R : class, new()
        {
            return _connection.Table<T>().Where(func);
        }
        public void Update(T t)
        {

            _connection.Update(t);
        }

        public void Close()
        {
            try
            {
                _connection.Close();
            }
            catch (Exception e)
            {
                throw new Exception(e.ToString());
            }
        }
        public void CreateDb(string dbPath)
        {
            _connection = new SQLiteConnection(dbPath, SQLiteOpenFlags.ReadWrite | SQLiteOpenFlags.Create);
        }
        public void DataService(string DatabaseName)
        {
#if UNITY_EDITOR
            var dbPath = string.Format("{0}/{1}", PublicClass.vesal_db_path, DatabaseName); //string.Format(@"Assets/StreamingAssets/{0}", DatabaseName);
#else
        

        var filepath = string.Format("{0}/{1}",PublicClass.vesal_db_path, DatabaseName);

       

        var dbPath = filepath;
        // var dbPath = loadDb;
#endif
            _connection = new SQLiteConnection(dbPath, SQLiteOpenFlags.ReadWrite | SQLiteOpenFlags.Create);
            //DebugLog.DebugLogInfo("Final PATH: " + dbPath);

        }

        public IEnumerable<T> Select(Func<T, bool> func)
        {
            throw new NotImplementedException();
        }
    }
}

```

> `RemoteRepository.cs` 接口类实现

```
using System;
using System.Collections.Generic;
using System.Text;
using Assets.Scripts.Infrastructure;
using UnityEngine;
using Assets.Scripts.Network;
using UnityEngine;
using Newtonsoft.Json;
namespace Assets.Scripts.Repository
{
    public class RemoteRepository<T> where T : class, new()
    {
        public ISerializer Serializer { get; set; }

        public RemoteRepository(ISerializer serializer = null)
        {
            Serializer = serializer ?? SerializerJson.Instance;
        }
        public void Get<R>(string url, T instance, Action<R> onSuccess) where R : class, new()
        {
            var parameters = HttpUtility.BuildParameters(instance, new StringBuilder("?"));
            var httpRequest = new HttpRequest
            {
                Url = url,
                Method = HttpMethod.Get,
                Parameters = parameters
            };
            Debug.Log(httpRequest.Url + httpRequest.Parameters);
            HttpClient.Instance.SendAsync(httpRequest, httpResponse =>
            {
                if (httpResponse.IsSuccess)
                {
                    R r = JsonConvert.DeserializeObject<R>(httpResponse.Data);
                    onSuccess(r);
                }
                //TODO:异常处理
            });
        }

        public void Get<R>(string url, Action<R> onSuccess) where R : class, new()
        {
            // var parameters = HttpUtility.BuildParameters(instance, new StringBuilder("?"));
            var httpRequest = new HttpRequest
            {
                Url = url,
                Method = HttpMethod.Get,
                // Parameters = parameters
            };
            HttpClient.Instance.SendAsync(httpRequest, httpResponse =>
            {
                if (httpResponse.IsSuccess)
                {
                    R r = JsonConvert.DeserializeObject<R>(httpResponse.Data);
                    Debug.Log("json str :" + r);
                    onSuccess(r);
                }
                //TODO:异常处理
            });
        }

        public void Post<R>(string url, T instance, Action<R> onSuccess) where R : class, new()
        {
            var parameters = HttpUtility.BuildParameters(instance, new StringBuilder());
            var httpRequest = new HttpRequest
            {
                Url = url,
                Method = HttpMethod.Post,
                Parameters = parameters
            };
            HttpClient.Instance.SendAsync(httpRequest, httpResponse =>
            {
                if (httpResponse.IsSuccess)
                {
                    //TODO:判断是否有Data
                    onSuccess(Serializer.Deserialize<R>(httpResponse.Data));
                }
            });
        }

        public void Test()
        {
            Debug.Log("Hello...");
        }
    }
}

```


> `Response.cs`



```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace Assets.Scripts.Infrastructure
{
    [System.Serializable]
    public class Response<T>
    {
        public string msg;
        public int code;
        public List<T> List;
        public string maxVersion;


    }

}

```

> `HttpUtility.cs` http操作类



```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace Assets.Scripts.Infrastructure
{
    public class HttpUtility
    {
        public static string BuildParameters<T>(T instance, StringBuilder sb) where T:class,new()
        {
            foreach (var property in instance.GetType().GetProperties())
            {
                var propertyName = property.Name;
                var value = property.GetValue(instance, null);
                sb.Append(propertyName + "=" + value + "&");
            }
            return sb.ToString().TrimEnd('&');
        }
    }
}

```

> `ISerializer.cs` 格式化接口 



```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace Assets.Scripts.Infrastructure
{
    public interface ISerializer
    {
        string Serialize<T>(T obj, bool readableOutput = false) where T : class, new();
        T Deserialize<T>(string json) where T : class, new();
    }
}

```

> `SerializerJson.cs` 



```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using UnityEngine;

namespace Assets.Scripts.Infrastructure
{
    public class SerializerJson:ISerializer
    {
        public static readonly SerializerJson Instance=new SerializerJson();
        private SerializerJson()
        {
            
        }
        public string Serialize<T>(T obj, bool readableOutput = false) where T : class, new()
        {
            throw new NotImplementedException();
        }

        public T Deserialize<T>(string json) where T : class, new()
        {
            return JsonUtility.FromJson<T>(json);
        }
    }
}

```



> `SQLite.cs`

[查看源码](https://gist.github.com/peterfei/69f98addc648163cd61ca63b9f53ef68)



> `HttpMethod.cs`

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace Assets.Scripts.Infrastructure
{
    public enum HttpMethod
    {
        Get,
        Post
    }
}

```


---
### Model实例 
> `TableVersions.cs`

```
using System;
using System.Collections.Generic;
using SQLite4Unity3d;

namespace Assets.Scripts.Model
{
    [Serializable]
    public class TableVersions
    {
        [PrimaryKey]
        public string table_name { get; set; }
        public string version { get; set; }
       
    }
}

```

> `GetStruct.cs` 

```
using System;


namespace Assets.Scripts.Model
{
    class GetStruct
    {
        public string Version { get; set; }
        public string UpdateUrlUuid { get; set; }
        public string device { get; set; }
        public string os { get; set; }
        public string level { get; set; }
        public string softVersion { get; set; }
    }
}

```


### 使用示例

```
 var cache_CommonLib = new CacheOption<GetStruct>();
 cache_CommonLib.setCache(PublicClass.server_ip+'v1/app/api')
```

