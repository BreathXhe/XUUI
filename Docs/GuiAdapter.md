# ��κ�����UI������

���������Ͼͱ���;���ĳ��UI����ϣ�ͨ��ʵ��һ��Adapter�Լ�һ��AdapterCollector�����ɺ�����UI����ϡ�

## Adapterʵ��

AdapterֻҪ�����������󼴿�

* �и�public string BindTo�ֶ�
* �������Ҫ����VM�仯����ʵ��DataConsumer�ӿڣ����Բ���ʽ����ʵ�֣�ֻҪ��DataConsumer�����Ľӿڼ��ɣ�
* �������Ҫ������ͬ����VM����ʵ��DataProducer�ӿ�
* �������Ҫ����һ���¼�����ʵ��EventEmitter�ӿ�

������˼·��
* ÿ��Adapter��MonoBehaviour��ֱ�Ӱ󶨵���Ӧ�ؼ��Ͻ��а���Ϣ������
* ÿ��Adapterֻ�Ǹ���ͨ����Ȼ����һ��MonoBehaviour��������UI����View���İ���Ϣ

��ugui��InputFieldΪ������������˼·��ʵ��

### ˼·1

�̳�AdapterBase<InputField>��AdapterBase<InputField>�̳���MonoBehaviour����BindTo�ֶΣ���ʵ�����Զ�����InputField�Ĺ��ܣ��������£�

~~~csharp
using UnityEngine;
using UnityEngine.UI;
using System;

namespace XUUI.UGUIAdapter
{
    public class InputFieldAdapter : AdapterBase<InputField>, DataConsumer<string>, DataProducer<string>
    {

        public Action<string> OnValueChange { get; set; } // InputField�����仯��Ҫ����OnValueChange

        public string Value // VM�����仯������õ���Setter����Ҫͬ����InputField
        {
            set
            {
                Target.text = value;
            }
        }

        void Start()
        {
            Target.onValueChange.AddListener((val) =>
            {
                if (OnValueChange != null)
                {
                    OnValueChange(val);
                }
            });
        }
    }
}
~~~

### ˼·2

�̳�RawAdapterBase��RawAdapterBaseֻ�Ǹ���ͨ���������˸�BindTo�ֶΣ����˶��ѣ��������£�

~~~csharp
using UnityEngine.UI;
using System;

namespace XUUI.UGUIAdapter
{
    public class RawInputFieldAdapter : RawAdapterBase, DataConsumer<string>, DataProducer<string>
    {
        private InputField target;

        public Action<string> OnValueChange { get; set; } // InputField�����仯��Ҫ����OnValueChange

        public string Value // VM�����仯������õ���Setter����Ҫͬ����InputField
        {
            set
            {
                target.text = value == null ? "" : value;
            }
        }

        public RawInputFieldAdapter(InputField input, string bindTo)
        {
            target = input;
            BindTo = bindTo;

            target.onValueChange.AddListener((val) =>
            {
                if (OnValueChange != null)
                {
                    OnValueChange(val);
                }
            });
        }
    }
}
~~~

һ��ͳһ�������࣬�������ð���Ϣ

~~~csharp
namespace XUUI.UGUIAdapter
{
    [Serializable]
    public class ButtonBindTo
    {
        public Button Target;
        public string BindTo;
    }

    [Serializable]
    public class TextBindTo
    {
        public Text Target;
        public string BindTo;
    }

    [Serializable]
    public class DropdownBindTo
    {
        public Dropdown Target;
        public string BindTo;
    }

    [Serializable]
    public class InputFieldBindTo
    {
        public InputField Target;
        public string BindTo;
    }

    public class BindToSetting : MonoBehaviour
    {
        public ButtonBindTo[] Buttons;

        public TextBindTo[] Texts;

        public DropdownBindTo[] Dropdowns;

        public InputFieldBindTo[] InputFields;
    }
}
~~~

## AdapterCollector

�ṩһ��luaģ�飬��һ��collect���������ܸ�UI�ڵ㣬����DataConsumer��DataProducer�Լ�EventEmitter

UGUI��ʵ���ǲ���C#��lua���ϵİ취��

lua����ܼ򵥣�ֱ�ӵ���C#��Ȼ�������ת��table

~~~lua
local _M = {}


function _M.collect(go)
    local infos = CS.XUUI.UGUIAdapter.Collector.Collect(go)
    local r = {}
    
    for i = 0, infos.Length - 1 do
        local objs = infos[i]
        local t = {}
        for j = 0, objs.Length - 1 do
            table.insert(t, objs[j])
        end
        table.insert(r, t)
    end
    
    return r
end

return _M
~~~

C#��Ҳ�����ӣ���Ӧ˼·1��Collector���£�

~~~csharp
using UnityEngine;
using System.Linq;

namespace XUUI.UGUIAdapter
{
    public class Collector
    {
        // [0]: DataConsumers
        // [1]: DataProducers
        // [2]: EventEmitters
        public static object[][] Collect(GameObject go)
        {
            var adapters = go.GetComponentsInChildren<AdapterBase>(true);

            var dataProducers = adapters
                .Where(adapter => adapter is DataProducer)
                .Select(o => (object)o)
                .ToArray();

            var dataConsumers = adapters
                .Where(adapter => adapter is DataConsumer)
                .Select(o => (object)o)
                .ToArray();

            var eventEmitters = adapters
                .Where(adapter => adapter is EventEmitter)
                .Select(o => (object)o)
                .ToArray();

            return new object[][] { dataConsumers, dataProducers, eventEmitters };
        }
    }
}
~~~

�����˼·2��Collect�߼����£�
~~~csharp
static object[][] collect(BindToSetting setting)
{
	var dataProducers = new List<object>();
	var dataConsumers = new List<object>();
	var eventEmitters = new List<object>();

	if (setting.InputFields != null)
	{
		var adpaters = setting.InputFields.Select(item => (object)new RawInputFieldAdapter(item.Target, item.BindTo));
		dataConsumers.AddRange(adpaters);
		dataProducers.AddRange(adpaters);

	}

	if (setting.Dropdowns != null)
	{
		var adpaters = setting.Dropdowns.Select(item => (object)new RawDropdownAdapter(item.Target, item.BindTo));
		dataConsumers.AddRange(adpaters);
		dataProducers.AddRange(adpaters);
	}

	if (setting.Texts != null)
	{
		dataConsumers.AddRange(setting.Texts.Select(item => (object)new RawTextAdapter(item.Target, item.BindTo)));
	}

	if (setting.Buttons != null)
	{
		eventEmitters.AddRange(setting.Buttons.Select(item => (object)new RawButtonAdapter(item.Target, item.BindTo)));
	}

	return new object[][] { dataConsumers.ToArray(), dataProducers.ToArray(), eventEmitters.ToArray() };
}
~~~
