# transfer-from-kml-to-csv
# python3
# -- coding: utf-8 --
# -------------------------------
# @Author : 郭健
# -------------------------------
# @File : kmltransfer.py
# @Software : PyCharm
# @Time : 2025/1/7 17:49
# -------------------------------
import xml.etree.ElementTree as ET
import csv

def extract_kml_data(kml_file):
    # 解析 KML 文件
    tree = ET.parse(kml_file)
    root = tree.getroot()

    # KML 命名空间
    namespace = {'kml': 'http://www.opengis.net/kml/2.2'}

    # 存储所有提取的数据
    data = []

    # 遍历所有 Placemark 节点
    for placemark in root.iter('{http://www.opengis.net/kml/2.2}Placemark'):
        row = {}

        # 提取 Name（如果存在）
        name = placemark.find('kml:name', namespace)
        row['Name'] = name.text if name is not None else ''

        # 提取 Description（如果存在）
        description = placemark.find('kml:description', namespace)
        row['Description'] = description.text if description is not None else ''

        # 提取 StyleUrl（如果存在）
        style_url = placemark.find('kml:styleUrl', namespace)
        row['Style URL'] = style_url.text if style_url is not None else ''

        # 提取 Coordinates（经纬度信息）
        coordinates = placemark.find('.//kml:Point/kml:coordinates', namespace)
        if coordinates is not None:
            coords = coordinates.text.strip()
            coords_list = coords.split(',')
            if len(coords_list) >= 2:
                row['Longitude'] = coords_list[0]
                row['Latitude'] = coords_list[1]
            else:
                row['Longitude'] = ''
                row['Latitude'] = ''
        else:
            row['Longitude'] = ''
            row['Latitude'] = ''

        # 提取其他子元素信息（如有）
        other_elements = placemark.findall('.//kml:*', namespace)
        for elem in other_elements:
            tag = elem.tag.split('}')[-1]  # 去除命名空间部分，获取标签名
            if tag not in row:  # 如果标签名未被提取，则添加它
                row[tag] = elem.text.strip() if elem.text else ''

        # 将提取的行添加到数据列表
        data.append(row)

    return data

def write_to_csv(data, csv_file):
    # 获取 CSV 文件的字段名称
    if len(data) > 0:
        fieldnames = data[0].keys()
    else:
        fieldnames = []

    # 写入 CSV 文件
    with open(csv_file, 'w', newline='', encoding='utf-8') as csvfile:
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        writer.writeheader()  # 写入标题行
        writer.writerows(data)  # 写入数据行

# 输入和输出文件路径
kml_file = r'C:\Users\miao\Desktop\质检量井.kml'  # 替换为你的 KML 文件路径
csv_file = r'D:\工作\output.csv'  # 替换为输出的 CSV 文件路径

# 提取 KML 数据
data = extract_kml_data(kml_file)

# 写入 CSV 文件
write_to_csv(data, csv_file)

print(f"数据已成功保存至 {csv_file}")





