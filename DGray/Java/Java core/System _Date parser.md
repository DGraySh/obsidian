---
title: 'System [[Date]] parser'
updated: 2021-03-11 07:57:43Z
created: 2021-03-11 07:56:49Z
tags:
  - date
---

System [[Date]] parser

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

public class DateUtil {

	// List of all date formats that we want to parse.
	// Add your own format here.
	private static List; 
			dateFormats = new ArrayList() {{
			add(new SimpleDateFormat("M/dd/yyyy"));
			add(new SimpleDateFormat("dd.M.yyyy"));
			add(new SimpleDateFormat("M/dd/yyyy hh:mm:ss a"));
			add(new SimpleDateFormat("dd.M.yyyy hh:mm:ss a"));
			add(new SimpleDateFormat("dd.MMM.yyyy"));
			add(new SimpleDateFormat("dd-MMM-yyyy"));
		}
	};

	/**
	 * Convert String with various formats into java.util.Date
	 * 
	 * @param input
	 *            Date as a string
	 * @return java.util.Date object if input string is parsed 
	 * 			successfully else returns null
	 */
	public static Date convertToDate(String input) {
		Date date = null;
		if(null == input) {
			return null;
		}
		for (SimpleDateFormat format : dateFormats) {
			try {
				format.setLenient(false);
				date = format.parse(input);
			} catch (ParseException e) {
				//Shhh.. try other formats
			}
			if (date != null) {
				break;
			}
		}

		return date;
	}
}
```