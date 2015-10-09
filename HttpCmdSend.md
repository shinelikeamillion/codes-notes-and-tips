```
package com.woapp.framework.conn;

import java.io.BufferedReader;
import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.util.Map;
import java.util.Map.Entry;
import java.util.Set;

import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.StatusLine;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.params.ClientPNames;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.params.CoreConnectionPNames;

import com.lidroid.xutils.http.client.multipart.HttpMultipartMode;
import com.lidroid.xutils.http.client.multipart.MultipartEntity;
import com.lidroid.xutils.http.client.multipart.content.FileBody;
import com.lidroid.xutils.http.client.multipart.content.StringBody;
import com.woapp.framework.conn.ICmd.CMD_TYPE;
import com.woapp.framework.conn.response.AbsCloudResponse;
import com.woapp.pubutils.JsonUtils;
import com.woapp.pubutils.WLog;
import com.woapp.system.AppContext;

public class HttpCmdSend {
	private static final String TAG = "HttpCmdSend";
	private static final String APPLICATION_JSON = "application/json";
	private static final String CONTENT_TYPE_TEXT_JSON = "text/json";

	/**
	 * 发送post消息
	 * 
	 * @param info
	 * @return
	 */
	public String sendPostCmd(CMD_TYPE cmd, String data) {
		WLog.e("postData", data);

		StringBuilder stringBuilder = new StringBuilder();
		// 组装url
		String url = getMethodBuilder(cmd);
		HttpClient client = new DefaultHttpClient();
		client.getParams().setParameter(ClientPNames.ALLOW_CIRCULAR_REDIRECTS,
				true);
		HttpPost httpPost = new HttpPost(url);
		// 增加basic 认证
		// HttpApiUtils.addBasic(httpPost);
		InputStream content = null;
		BufferedReader reader = null;
		String ret = "";
		try {
			// 1,把data转到map 2,给表单添加数据
			httpPost.setEntity(postMethodParamBUilder(JsonUtils
					.jsonStrToMap(data)));

			HttpResponse response = client.execute(httpPost);

			StatusLine statusLine = response.getStatusLine();
			int statusCode = statusLine.getStatusCode();

			if (statusCode == 200) {
				HttpEntity entity = response.getEntity();
				content = entity.getContent();
				reader = new BufferedReader(new InputStreamReader(content));
				String line;
				while ((line = reader.readLine()) != null) {
					stringBuilder.append(line);
				}
				ret = stringBuilder.toString();
			} else {
				HttpEntity entity = response.getEntity();
				content = entity.getContent();
				reader = new BufferedReader(new InputStreamReader(content));
				String line;
				while ((line = reader.readLine()) != null) {
					stringBuilder.append(line);
				}
				// Log.e(TAG, stringBuilder.toString());
			}
		} catch (ClientProtocolException e) {
			AbsCloudResponse absCloudResponse = new AbsCloudResponse();
			absCloudResponse.setResult("-01");
			e.printStackTrace();
			return absCloudResponse.getJson();
		} catch (IOException e) {
			AbsCloudResponse absCloudResponse = new AbsCloudResponse();
			absCloudResponse.setResult("-02");
			e.printStackTrace();
			return absCloudResponse.getJson();
		}
			finally {
			if (reader != null) {
				try {
					reader.close();
				} catch (IOException e) {
					AbsCloudResponse absCloudResponse = new AbsCloudResponse();
					absCloudResponse.setResult("-03");
					e.printStackTrace();
					return absCloudResponse.getJson();
				}
			}
			if (content != null) {
				try {
					content.close();
				} catch (IOException e) {
					AbsCloudResponse absCloudResponse = new AbsCloudResponse();
					absCloudResponse.setResult("-04");
					e.printStackTrace();
					return absCloudResponse.getJson();
				}
			}
		}

		WLog.e(TAG, "url:" + url + ";result:" + stringBuilder.toString());
		// TODO 远程调用出错时，自定义返回值
		WLog.e("postRet", ret);
		return ret;
	}

	// 获取url ：不包含GET的参数
	private String getMethodBuilder(CMD_TYPE cmd) {
		String serverAddr = "";

		int type = cmd.type;
		switch (type) {
		case ICmd.TYPE_CLOUD: {
			serverAddr = AppContext.getInstance().getConnInfo()
					.getCmdServerCloud();
			break;
		}
		case ICmd.TYPE_CLOUD2: {

			serverAddr = AppContext.getInstance().getConnInfo()
					.getCmdServerCloud2();
			break;
		}

		default: {
			break;
		}
		}

		String tmp = AppContext.getInstance().getConnInfo().getPrefix() + "://"
				+ serverAddr + cmd.cgi;// http://
		return tmp;
	}

	private String getMethodParamBuilder(String url, Map<String, String> params) {
		// 参数不为空则拼接参数
		if (params != null) {
			Set<Entry<String, String>> sets = params.entrySet();
			StringBuilder sb = new StringBuilder();
			for (Entry<String, String> entry : sets) {
				String key = entry.getKey();
				String value = entry.getValue();
				if (!key.isEmpty() && !value.isEmpty()) {
					sb.append("&" + key + "=" + value);
				}
			}
			url = url + sb.toString();
		}
		return url;
	}

	/**
	 * 补充post参数
	 */
	private MultipartEntity postMethodParamBUilder(Map<String, String> params) {

		MultipartEntity multipartEntity = new MultipartEntity(
				HttpMultipartMode.BROWSER_COMPATIBLE);
		// 参数不为空则拼接参数
		if (params != null) {
			Set<Entry<String, String>> sets = params.entrySet();
			for (Entry<String, String> entry : sets) {
				String key = entry.getKey();
				String value = entry.getValue();
				if (!key.isEmpty() && !value.isEmpty()) {

					// 如果有头像则添加文件到表单
					if (key.equals("image")) {
						File file = new File(value);
						if (file.exists() && file.isFile()) {
							FileBody fileBody = new FileBody(file);
							multipartEntity.addPart("image", fileBody);
						}
					} else {
						try {
							multipartEntity.addPart(key, new StringBody(value));
						} catch (UnsupportedEncodingException e) {
							e.printStackTrace();
						}
					}
				}
			}
		}
		return multipartEntity;
	}

	/**
	 * 发送get消息
	 * 
	 * @param info  这个方法现在没有用到
	 * @return
	 */
	public String sendGetCmd(CMD_TYPE cmd, Map<String, String> params) {
		// 用户在发送请求失败或者出现异常的时候返回错误值
		AbsCloudResponse absCloudResponse = new AbsCloudResponse();
		
		HttpClient client = new DefaultHttpClient();
		client.getParams().setParameter(ClientPNames.ALLOW_CIRCULAR_REDIRECTS,
				true);
		client.getParams().setParameter(
				CoreConnectionPNames.CONNECTION_TIMEOUT, 5000);
		client.getParams().setParameter(CoreConnectionPNames.SO_TIMEOUT, 15000);
		// 拼接get cgi的url 的 http+服务器域名+action
		String url = getMethodBuilder(cmd);
		// 补充get参数
		url = getMethodParamBuilder(url, params);
		WLog.e(TAG, "GET url:" + url);
		HttpGet get = new HttpGet(url);
		get.addHeader("Charset", "UTF-8");
		get.addHeader("Content-Type", "application/json");
		get.addHeader("Accept", "application/json");
		InputStream content = null;
		BufferedReader reader = null;
		StringBuilder stringBuilder = new StringBuilder();
		String retString = "";// gson.toJson(def);
		try {
			HttpResponse response = client.execute(get);

			StatusLine statusLine = response.getStatusLine();
			int statusCode = statusLine.getStatusCode();

			if (statusCode == 200) {
				HttpEntity entity = response.getEntity();
				content = entity.getContent();
				reader = new BufferedReader(new InputStreamReader(content));
				String line;
				while ((line = reader.readLine()) != null) {
					stringBuilder.append(line);
				}
				retString = stringBuilder.toString();
			} else {
				WLog.e("JSON", "Failed to download file");
				absCloudResponse.setErrCode("-01");
				return absCloudResponse.getJson();
			}
		} catch (ClientProtocolException e) {
			e.printStackTrace();
			
			return retString;
		} catch (IOException e) {
			e.printStackTrace();
			return retString;
		} finally {
			if (reader != null) {
				try {
					reader.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			if (content != null) {
				try {
					content.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			// return retString;
		}
		return retString;
	}

	/**
	 * 发送get消息
	 * 
	 * @param info
	 * @return
	 */
	public String sendGetCmd(CMD_TYPE cmd, String paramStr) {
		
		System.out.println("sendGetCmd==========================="+paramStr);
		
		AbsCloudResponse responseJson = new AbsCloudResponse();

		HttpClient client = new DefaultHttpClient();
		// TODO 重定向
		client.getParams().setParameter(ClientPNames.ALLOW_CIRCULAR_REDIRECTS,
				true);
		// 拼接get cgi的url 的 http+服务器域名+action
		String url = getMethodBuilder(cmd);
		WLog.e(TAG, "GET url:" + url);
		StringBuilder sb = new StringBuilder();
		sb.append(url);
		if (paramStr != null && !paramStr.trim().isEmpty()) {

			WLog.e(TAG, "GET param:" + paramStr);
			String encode = URLEncoder.encode(paramStr);
			sb.append("?param=");
			sb.append(encode);
		}

		HttpGet get = new HttpGet(sb.toString());
		// get.addHeader("Charset", "UTF-8");
		// get.addHeader("Content-Type", "application/json");
		// get.addHeader("Accept", "application/json");
		InputStream content = null;
		BufferedReader reader = null;
		StringBuilder stringBuilder = new StringBuilder();
		String retString = "";
		try {
			HttpResponse response = client.execute(get);

			StatusLine statusLine = response.getStatusLine();
			int statusCode = statusLine.getStatusCode();

			if (statusCode == 200) {
				HttpEntity entity = response.getEntity();
				content = entity.getContent();
				reader = new BufferedReader(new InputStreamReader(content));
				String line;
				while ((line = reader.readLine()) != null) {
					stringBuilder.append(line);
				}
				retString = stringBuilder.toString();
				System.out.println("request: " + retString);
				// WLog.e(TAG, "request: " + retString);
			} else {
				WLog.e(TAG, "request error code: " + statusCode);
				responseJson.setErrCode("-005");
				responseJson.setErrDesc(statusCode + "");
			}
		} catch (ClientProtocolException e) {
			responseJson.setErrCode("-001");
			responseJson.setErrDesc(e.getMessage());
			WLog.e(TAG, "ClientProtocolException " + e.getMessage());
		} catch (IOException e) {
			responseJson.setErrCode("-002");
			responseJson.setErrDesc(e.getMessage());
			WLog.e(TAG, "IOException " + e.getMessage());
		} finally {

			if (reader != null) {
				try {
					reader.close();
				} catch (IOException e) {
					responseJson.setErrCode("-003");
					responseJson.setErrDesc(e.getMessage());
					WLog.e(TAG, "IOException " + e.getMessage());
				}
			}
			if (content != null) {
				try {
					content.close();
				} catch (IOException e) {
					responseJson.setErrCode("-004");
					responseJson.setErrDesc(e.getMessage());
					WLog.e(TAG, "IOException " + e.getMessage());
				}
			}

			if (retString.isEmpty()) {
				retString = responseJson.getJson();
			}
		}
		System.out.println("sendResult==================================="+retString);
		return retString;
	}
}

```
