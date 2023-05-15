# thiyaku.search

import { StyleSheet, Text, View, TouchableOpacity, Animated, TextInput, Dimensions, ScrollView, TouchableNativeFeedback, ActivityIndicator, Modal, FlatList } from 'react-native'
import React from 'react'
import LocalConfig from '../LocalConfig'
import Ionicons from 'react-native-vector-icons/Ionicons';
import { List, Button, RadioButton, FAB } from 'react-native-paper';
import CheckBox from '@react-native-community/checkbox';
import Entypo from 'react-native-vector-icons/Entypo';
import { useRef } from 'react';
import { useState } from 'react';
import { useEffect } from 'react';
import { Image } from 'react-native';
import { Grayscale } from 'react-native-color-matrix-image-filters';
import MaterialIcons from 'react-native-vector-icons/MaterialIcons';
import sqlservice from './sql';
import { postData } from '../Functions';
import { BlurView } from '@react-native-community/blur';
import { useDispatch, useSelector } from 'react-redux';
import {
  addAllItems,
  addFreeItem,
  addToCart,
  checkItem,
  deleteItem,
  reduxAddaddonsItem,
  removeromCart,
} from './Redux/Actions';
import { useNavigation } from '@react-navigation/native';
import { BackHandler } from 'react-native';
const sql = new sqlservice();
const menu_item_icon_IMAGE = `${LocalConfig.API_URL}admin/images/menu_item_icon/`;
const SearchScreen = ({ route }) => {
  // console.log(JSON.stringify())
  // const initalItems = useSelector(state => state.items);
  const initalItems = route.params.data;
  const navigation = useNavigation();
  const [searchQuery, setSearchQuery] = useState("")
  const [allMenuFromRedux, setAllMenuFromRedux] = useState([])
  const [wholeCat3Input, setWholeCat3Input] = useState(0);
  const [itemModelItem, setItemModelItem] = useState({
    item: undefined,
    catIndex: undefined,
    itemIndex: undefined,
    quantity: undefined,
  });
  const [itemModel, setItemModel] = useState(false);
  const [refresh, setRefresh] = useState(false);
  const [loadingItem, setLoadingItem] = useState(null);
  const [addonsModal, setAddonsModal] = useState(false);
  const [repeatModal, setRepeatModal] = useState(true);
  const [repeatModalItem, setRepeatModalItem] = useState({
    item: undefined,
    catIndex: undefined,
    itemIndex: undefined,
    price: undefined,
  });
  const [addonsModalItem, setAddonsModalItem] = useState({
    item: undefined,
    catIndex: undefined,
    itemIndex: undefined,
    variant: undefined,
    ingredient: undefined,
    totel: undefined,
  });
  const [addonsSeleted, setAddonsSeleted] = useState({
    radioButton: { price: [], id: [], name: [] },
    CheckBoxes: { price: [], id: [], name: [] },
    ingrediant: { price: [], id: [], name: [] },
  });
  const [showMore, setShowMore] = useState({ id: null, description: null });
  // COUPON state starts
  const [couponInfoItem, setCouponInfoItem] = useState({})
  const [couponInfoModal, setCouponInfoModal] = useState(false)
  const [meatInput, setMeatInput] = useState(0)
  const [meatInputModal, setMeatInputModal] = useState(false)
  const [meatInputModalItem, setMeatInputModalItem] = useState({
    item: {},
    catIndex: undefined,
    itemIndex: undefined,
  })
  const dispatch = useDispatch();
  const getTotelPrice = state => state.TotelPrice;
  const searchdata = ({ Query = "" }) => {
    setSearchQuery(Query)
    const filteredItems = initalItems.map((catItem, catIndex) => {
      const filteredData = catItem.data
        .map((item, itemIndex) => ({
          ...item,
          catIndex,
          itemIndex
        }))
        .filter(item =>
          item.menu_name.toLowerCase().includes(Query.toLowerCase())
        );
      return { ...catItem, data: filteredData };
    });
    if (Query == "") {
      setAllMenuFromRedux([]);
    } else
      setAllMenuFromRedux(filteredItems);
  };


  const TotelPrice = useSelector(getTotelPrice) || { items: 0, price: 0 };
  const openCouponInfoModal = ({ item = {}, canIOpen = false }) => {
    if (canIOpen) {
      setCouponInfoModal(true)
      setCouponInfoItem(item)
    } else {
      setCouponInfoModal(false)
      setCouponInfoItem({})
    }
  }
  const refreshME = () => {
    setRefresh(!refresh);
    searchdata({ Query: searchQuery })
  };
  const addaddonsItem = ({
    catIndex,
    item,
    itemIndex,
    addonsSeleted,
    qty,
    price,
  }) => {
    dispatch(
      reduxAddaddonsItem({
        catIndex,
        item: item,
        itemIndex,
        addonsSeleted,
        qty,
        price,
      }),
    );
    setAddonsSeleted({
      radioButton: { price: [], id: [], name: [] },
      CheckBoxes: { price: [], id: [], name: [] },
      ingrediant: { price: [], id: [], name: [] },
    });
    setAddonsModalItem({
      item: undefined,
      catIndex: undefined,
      itemIndex: undefined,
      variant: undefined,
      totel: undefined,
    });
    setItemModel(false);
    setLoadingItem(null);
    setAddonsModal(true);
    refreshME();
  };
  const addingItem = async ({
    catIndex,
    item,
    itemIndex,
    price,
    repeat = false,
  }) => {
    const ADDItem = ({ qty, type, isFree = false }) => {
      dispatch(addToCart([catIndex, itemIndex, qty, parseInt(price), item, type, isFree]));
      refreshME();
      setAddonsSeleted({
        radioButton: { price: [], id: [], name: [] },
        CheckBoxes: { price: [], id: [], name: [] },
        ingrediant: { price: [], id: [], name: [] },
      });
      setAddonsModalItem({
        item: undefined,
        catIndex: undefined,
        itemIndex: undefined,
        variant: undefined,
        ingredient: undefined,
        totel: undefined,
      });
      setAddonsModal(false);
      setLoadingItem(null);
      itemModelItem.item.qty = qty;
      setItemModelItem({
        catIndex: itemModelItem.catIndex,
        item: itemModelItem.item,
        itemIndex: itemModelItem.itemIndex,
        quantity: qty,
      });
      setMeatInput(0)
      setMeatInputModal(false)
    };
    var qty;
    if (item.ingrecount > 0) {
      setLoadingItem(item.id);
      sql.getItem(item.id).then(async res => {
        if (res.rows.length > 0 && !repeat) {
          setRepeatModal(true);
          setRepeatModalItem({
            catIndex,
            item,
            itemIndex,
            price,
          });
          setLoadingItem(false);
        } else {
          const ADD_ONS_ITEM_API = `${LocalConfig.API_URL}/admin/api/ingredientsnew.php?category=${item.category}&menu_id=${item.id}`;
          await postData(ADD_ONS_ITEM_API).then(res => {
            var myObj = { id: [], price: [], name: [] };
            res.variant.map(eachVariant => {
              if (eachVariant.data.length > 0) {
                if (eachVariant.status == 1 && eachVariant.data[0].id) {
                  let pushed = false
                  eachVariant.data.map((variantItem, variantIndex) => {
                    if (variantItem.selection == 1) {
                      myObj.id.push(eachVariant.data[variantIndex].id);
                      myObj.price.push(eachVariant.data[variantIndex].price);
                      myObj.name.push(eachVariant.data[variantIndex].variant_name);
                      pushed = !pushed
                    }
                  })
                  if (!pushed) {
                    myObj.id.push(eachVariant.data[0].id);
                    myObj.price.push(eachVariant.data[0].price);
                    myObj.name.push(eachVariant.data[0].variant_name);
                  }
                }
              }
            });
            var totel = parseFloat(item.price);
            myObj.price.map(radioButtonPrice => {
              if (radioButtonPrice != '' && !isNaN(parseFloat(radioButtonPrice))) totel += parseFloat(radioButtonPrice);
            });
            setAddonsSeleted({
              radioButton: myObj,
              CheckBoxes: { id: [], price: [], name: [] },
              ingrediant: { price: [], id: [], name: [] },
            });
            setAddonsModalItem({
              item: item,
              catIndex,
              itemIndex,
              variant: res.variant,
              ingredient: res.ingre,
              totel: totel,
            });
            // setItemModel(true);
            setLoadingItem(null);
            setAddonsModal(true);
          });
        }
      });
    } else {
      sql.getItem(item.id).then(res => {
        if (res.rows.length <= 0) {
          console.log({
            catIndex,
            itemIndex
          })
          if (
            item.wholecat == 5 ||
            item.wholecat == 1
          ) {
            qty = parseInt(item.qty) + 1;
            ADDItem({ qty, type: 'add' });
          }
          else if (item.wholecat == 2) {
            qty =
              parseInt(item.qty) + 0.25;
            ADDItem({ qty, type: 'add' });
          }
          else if (item.wholecat == 3) {
            if (!meatInputModal) {
              setMeatInput(item.qty)
              setMeatInputModal(true)
              setMeatInputModalItem({
                item: item,
                catIndex,
                itemIndex,
              })
            } else {
              ADDItem({ qty: parseFloat(meatInput), type: 'add' });
            }
          }
        } else if (res.rows.length > 0) {
          if (
            item.wholecat == 5 ||
            item.wholecat == 1
          ) {
            qty = parseInt(item.qty) + 1;
            ADDItem({ qty: qty, type: 'update' });
          } else if (item.wholecat == 2) {
            qty =
              parseFloat(item.qty) + 0.25;
            ADDItem({ qty: qty, type: 'update' });
          } else if (item.wholecat == 3) {
            if (!meatInputModal) {
              setMeatInput(item.qty)
              setMeatInputModal(true)
              setMeatInputModalItem({
                item: item,
                catIndex,
                itemIndex,
              })
            } else {
              ADDItem({ qty: meatInput, type: 'update' });
            }
          }
        }
      });
    }
  };
  const openItemModal = ({ item, disable = false, modal = true }) => {
    if (disable) {
      setAddonsModalItem({
        item: undefined,
        catIndex: undefined,
        itemIndex: undefined,
        variant: undefined,
        totel: undefined,
      });
      setItemModel(false);
    } else {
      for (let catIndex = 0; catIndex < allMenuFromRedux.length; catIndex++) {
        for (
          let itemIndex = 0;
          itemIndex < allMenuFromRedux[catIndex].data.length;
          itemIndex++
        ) {
          if (item.id == item.id) {
            item.sqlid = item.sqlid;
            setItemModelItem({
              item,
              catIndex,
              itemIndex,
              quantity: item.qty,
            });
            if (modal) setItemModel(true);
          }
        }
      }
    }
  };
  const removingItem = ({ catIndex, item, itemIndex, price }) => {
    const MinusItem = ({ qty }) => {
      dispatch(removeromCart([catIndex, itemIndex, qty, price, item]));
      refreshME();
      setLoadingItem(null);
      setItemModelItem({
        catIndex: itemModelItem.catIndex,
        item: itemModelItem.item,
        itemIndex: itemModelItem.itemIndex,
        quantity: qty,
      });
      openItemModal({ item: item, modal: false });
    };
    var qty;
    if (item.wholecat == 5 || item.wholecat == 1) {
      qty = parseInt(item.qty) - 1;
    }
    else if (item.wholecat == 2)
      qty =
        parseFloat(item.qty) - 0.25;
    if (qty > 0) {
      sql.getItem(item.id).then(res => {
        if (res.rows.length <= 1) {
          MinusItem({ qty });
        } else if (res.rows.length > 0) {
          Alert.alert(
            'Alert', 'You have added more then one item, Please update from your cart page.',
          );
        }
      });
    } else {
      dispatch(deleteItem([catIndex, itemIndex, 0, price, item]));
    }

    setTimeout(() => {
      setLoadingItem(null);
      refreshME();
    }, 50);
  };
  const handleBackPress = () => {
    // setAllMenuFromRedux([])
    navigation.goBack()
    searchdata({ Query: "" })
    return true; // Prevent default back button behavior
  }
  useEffect(() => {
    const backHandler = BackHandler.addEventListener('hardwareBackPress', () => handleBackPress());
    return () => backHandler.remove();
  }, []);
  useEffect(() => searchdata({ Query: searchQuery }), [refresh])
  return (
    <View style={styles.container}>
      {/* Seach bar starts */}
      <View
        style={{
          flexDirection: 'row',
          backgroundColor: LocalConfig.COLOR.BLACK,
          alignItems: 'center',
          paddingVertical: 5,
        }}>
        <TouchableOpacity
          style={{
            flex: 1,
            justifyContent: 'center',
            alignItems: 'center',
          }}
          onPress={handleBackPress}>
          <Ionicons
            name="arrow-back"
            color={LocalConfig.COLOR.UI_COLOR}
            size={28}
          />
        </TouchableOpacity>
        <View style={{
          flex: 8,
          paddingVertical: 5,
          flexDirection: 'row'
        }}>
          <TextInput
            style={styles.input}
            onChangeText={(Query) => {
              searchdata({ Query })
            }}
            value={searchQuery}
            placeholder="Search for more.."
            placeholderTextColor={LocalConfig.COLOR.BLACK_LIGHT}
            keyboardType="default"
            clearButtonMode="while-editing"
            numberOfLines={1}
          />
          {searchQuery.length > 0 ? <View
            // activeOpacity={1}
            style={styles.clearSearch}
          >
            <TouchableOpacity
              onPress={() => searchdata({ Query: "" })}
            >
              <Ionicons
                name="close-circle"
                color={LocalConfig.COLOR.UI_COLOR}
                size={28}
              />
            </TouchableOpacity>
          </View> : <View style={styles.clearSearch} />}
        </View>
        <View style={{ flex: 0.2 }} />
      </View>
      {/* Seach bar ends */}
      {/* search items starts */}
      <ScrollView keyboardShouldPersistTaps='handled' style={styles.search_items_view}>
        {allMenuFromRedux.map((catItem, catIndex) => catItem.data.map((item, index) => <View
          key={index}
          style={{
            flexDirection: 'row',
            alignItems: 'center',
            margin: 15,
            height:
              Math.round(Dimensions.get('screen').height) /
              6.5,
          }}>
          <View
            style={{
              flex: 2,
            }}>
            <View
              style={{
                alignItems: 'center',
                flexDirection: 'row',
              }}>
              <Image
                style={{
                  width: 15,
                  height: 15,
                  marginRight: 5,
                  backgroundColor: LocalConfig.COLOR.WHITE,
                }}
                resizeMode={'cover'}
                source={
                  (item.food_type == 2)
                    ? require('../assests/non_veg.png')
                    : require('../assests/veg.png')
                }
              />
              <Text
                style={{
                  color: LocalConfig.COLOR.WHITE,
                  fontSize: 14,
                  fontFamily: 'verdanab',
                }}>
                {item.menu_name}
              </Text>
            </View>
            <View>
              <Text
                style={{
                  color: LocalConfig.COLOR.WHITE,
                  paddingTop: 8,
                  fontFamily: 'Proxima Nova Font',
                }}>
                {`\u20B9 ${item.price || item.dprice}`}
              </Text>
              {item.description != '' &&
                (showMore.id != item.id ? (
                  <Text
                    numberOfLines={2}
                    onPress={e => {
                      setShowMore({
                        id: item.id,
                        description: item.description,
                      });
                    }}
                    style={{
                      color: LocalConfig.COLOR.WHITE_LIGHT,
                      paddingVertical: 8,
                      fontFamily: 'Proxima Nova Font',
                      textAlign: 'justify',
                      paddingRight: 8,
                    }}>
                    {item.description}
                  </Text>
                ) : (
                  <Text
                    onPress={e => {
                      setShowMore({
                        id: null,
                        description: null,
                      });
                    }}
                    style={{
                      color: LocalConfig.COLOR.BLACK_LIGHT,
                      paddingVertical: 8,
                      fontFamily: 'Proxima Nova Font',
                      textAlign: 'justify',
                      paddingRight: 8,
                    }}>
                    {showMore.description}
                  </Text>
                ))}
            </View>

          </View>
          <View
            style={{
              flex: 1,
              justifyContent: 'center',
              alignItems: 'center',
            }}>
            {item.status == 0 && item.menu_image != '' ? (
              <Grayscale>
                <TouchableOpacity
                  disabled={true}
                >
                  <Image
                    onError={error => {
                      item.menu_image = 'NoImage.png';
                    }}
                    style={{
                      width:
                        (Dimensions.get('window').width /
                          100) *
                        30,
                      height:
                        (Dimensions.get('window').width /
                          100) *
                        25,
                      borderRadius: 5,
                    }}
                    source={{
                      uri:
                        menu_item_icon_IMAGE +
                        item.menu_image,
                    }}
                    resizeMode="cover"
                  />
                </TouchableOpacity>
              </Grayscale>
            ) : (
              item.menu_image != '' && (
                <TouchableOpacity
                  disabled={true}
                  onPress={() => openItemModal({ item })}>
                  <Image
                    onError={error => {
                      item.menu_image = 'NoImage.png';
                    }}
                    style={{
                      width:
                        (Dimensions.get('window').width /
                          100) *
                        30,
                      height:
                        (Dimensions.get('window').width /
                          100) *
                        25,
                      borderRadius: 5,
                    }}
                    source={{
                      uri:
                        menu_item_icon_IMAGE +
                        item.menu_image,
                    }}
                    resizeMode="cover"
                  />
                </TouchableOpacity>
              )
            )}
            <View
              style={{
                backgroundColor: item.status == 0 ? LocalConfig.COLOR.UI_COLOR_LITE_TWICE : LocalConfig.COLOR.UI_COLOR,
                width:
                  Math.round(
                    Dimensions.get('screen').width,
                  ) / 4,
                marginTop: item.status != 0 ? '-10%' : 0,
                padding: 5,
                borderRadius: 5,
                elevation: 20,
                justifyContent: 'center',
                alignSelf: 'center',
              }}>
              {item.status == 0 ? (
                item.timing_state == 0 ? (
                  <Text
                    style={{
                      color: LocalConfig.COLOR.BLACK,
                      fontFamily: 'Proxima Nova Bold',
                      textAlign: 'center',
                    }}>
                    SOLD
                  </Text>
                ) : (
                  <Text
                    style={{
                      color: LocalConfig.COLOR.BLACK,
                      fontFamily: 'Proxima Nova Bold',
                      textAlign: 'center',
                      fontSize: 12
                    }}>{`Next available at ${item.start}`}</Text>
                )
              ) : item.qty <= 0 ? (
                <TouchableOpacity
                  onPress={() => {
                    if (loadingItem != item.id)
                      addingItem({
                        catIndex: item.catIndex,
                        item,
                        itemIndex: item.itemIndex,
                        price: item.dprice ? item.dprice : item.price,
                      });
                  }}
                  style={{
                    flexDirection: 'row',
                    justifyContent: 'center',
                    alignItems: 'center',
                    width: '100%',
                  }}>
                  <Text
                    style={{
                      color: LocalConfig.COLOR.BLACK,
                      fontFamily: 'Proxima Nova Bold',
                      fontSize: 18,
                    }}>
                    {loadingItem != item.id ? (
                      'ADD'
                    ) : (
                      <ActivityIndicator
                        size={
                          Math.round(
                            Dimensions.get('screen').width,
                          ) / 20
                        }
                        color={LocalConfig.COLOR.BLACK}
                      />
                    )}
                  </Text>
                </TouchableOpacity>
              ) : (
                <View
                  style={{
                    flexDirection: 'row',
                    justifyContent: 'space-around',
                    alignItems: 'center',
                  }}>
                  {(loadingItem != item.id && item.wholecat != 3) && (
                    <MaterialIcons
                      name="remove"
                      size={20}
                      color={LocalConfig.COLOR.BLACK}
                      onPress={() => {
                        if (loadingItem != item.id)
                          removingItem({
                            catIndex: item.catIndex,
                            item,
                            itemIndex: item.itemIndex,
                            price: item.price,
                          });
                      }}
                    />
                  )}
                  {(loadingItem != item.id && item.wholecat == 3 && item.ingrecount <= 0) && (
                    <MaterialIcons
                      name="edit"
                      size={20}
                      color={LocalConfig.COLOR.BLACK}
                      onPress={() => {
                        if (loadingItem != item.id) {
                          addingItem({
                            catIndex: item.catIndex,
                            item,
                            itemIndex: item.itemIndex,
                            price: item.price || item.dprice,
                          })
                        }
                      }}
                    />
                  )}
                  <Text
                    style={{
                      color: LocalConfig.COLOR.BLACK,
                      fontFamily: 'Proxima Nova Bold',
                      fontSize: 18,
                      textAlign: 'center',
                    }}>
                    {/* {item.qty} */}
                    {loadingItem != item.id ? (
                      item.qty
                    ) : (
                      <ActivityIndicator
                        size={
                          Math.round(
                            Dimensions.get('screen').width,
                          ) / 20
                        }
                        color={LocalConfig.COLOR.BLACK}
                      />
                    )}
                  </Text>
                  {(loadingItem != item.id && item.wholecat != 3) && (
                    <MaterialIcons
                      name="add"
                      size={20}
                      color={LocalConfig.COLOR.BLACK}
                      onPress={() => {
                        if (loadingItem != item.id)
                          addingItem({
                            catIndex: item.catIndex,
                            item,
                            itemIndex: item.itemIndex,
                            price: item.price || item.dprice,
                          });
                      }}
                    />
                  )}
                  {(loadingItem != item.id && item.wholecat == 3) && (
                    <MaterialIcons
                      name="delete"
                      size={20}
                      color={LocalConfig.COLOR.BLACK}
                      onPress={() => {
                        if (loadingItem != item.id) {
                          item.qty = 0
                          removingItem({
                            catIndex: item.catIndex,
                            item,
                            itemIndex: item.itemIndex,
                            price: item.price,
                          });

                        }
                      }}
                    />
                  )}
                </View>
              )}
            </View>
            {item.ingrecount > 0 && (
              <Text
                style={{
                  color: LocalConfig.COLOR.BLACK_LIGHT,
                }}>
                customizable
              </Text>
            )}
          </View>
        </View>)
        )}
      </ScrollView>
      {/* search items ends */}
      {/* VIEW CART STARTS */}
      {TotelPrice.items > 0 && TotelPrice.price > 0 && (
        <TouchableOpacity
          activeOpacity={0}
          onPress={() => navigation.navigate('CartScreen')}
          style={{
            backgroundColor: LocalConfig.COLOR.UI_COLOR,
            height: (Dimensions.get('window').height / 100) * 7,
            display: 'flex',
            // justifyContent: 'center',
            flexDirection: 'row',
            alignItems: 'center',
          }}>
          <View
            style={{ flex: 2, marginHorizontal: 10, alignItems: 'flex-start' }}>
            <Text
              style={{ fontFamily: 'verdanab', color: LocalConfig.COLOR.BLACK }}>
              TOTEL ITEMS
              {` ${TotelPrice.items} | \u20B9 ${TotelPrice.price}`}
            </Text>
          </View>
          <View style={{ flex: 1, marginHorizontal: 10, alignItems: 'flex-end' }}>
            <Text
              style={{ fontFamily: 'verdanab', color: LocalConfig.COLOR.BLACK }}>
              VIEW CART
            </Text>
          </View>
        </TouchableOpacity>
      )}
      {/* VIEW CART ENDS */}
      {itemModelItem.item && (
        <Modal
          transparent={true}
          visible={itemModel}
          onBackdropPress={() => openItemModal({ disable: true })}
          onRequestClose={() => openItemModal({ disable: true })}
          animationType={'slide'}
          style={{
            justifyContent: 'flex-end',
            alignItems: 'center',
            flex: 1,
            width: '100%',
            margin: 0,
            backgroundColor: 'rgba(0,0,0,0.2)',
            height: Dimensions.get('window').height / 2,
          }}>
          <View style={{ flex: 1 }} />
          <View>
            <View
              style={{
                flexDirection: 'row',
                height: (Dimensions.get('window').height / 100) * 7,
                backgroundColor: LocalConfig.COLOR.UI_COLOR,
                alignItems: 'center',
                width: Dimensions.get('window').width,
                justifyContent: 'flex-end',
                borderTopLeftRadius: 20,
                borderTopRightRadius: 20,
              }}>
              <View style={{ flex: 1 }} />
              <Text
                style={{
                  flex: 10,
                  textAlign: 'center',
                  fontFamily: 'verdanab',
                  color: LocalConfig.COLOR.BLACK,
                }}>
                {`${itemModelItem.item.menu_name}`.toUpperCase()}
              </Text>
              <TouchableOpacity
                style={{ flex: 1 }}
                onPress={() => openItemModal({ disable: true })}>
                <MaterialIcons
                  name="close"
                  size={25}
                  color={LocalConfig.COLOR.BLACK}
                />
              </TouchableOpacity>
            </View>
            <View>
              <View>
                {itemModelItem.item.status == 0 ?
                  <Grayscale>
                    <Image
                      source={{
                        uri: menu_item_icon_IMAGE + itemModelItem.item.menu_image,
                      }}
                      style={{
                        width: Dimensions.get('window').width,
                        height: Dimensions.get('window').width / 1.5,
                      }}
                      resizeMode={'stretch'}
                    />
                  </Grayscale>
                  :
                  <Image
                    source={{
                      uri: menu_item_icon_IMAGE + itemModelItem.item.menu_image,
                    }}
                    style={{
                      width: Dimensions.get('window').width,
                      height: Dimensions.get('window').width / 1.5,
                    }}
                    resizeMode={'stretch'}
                  />
                }
              </View>
              <View
                style={{
                  flexDirection: 'row',
                  backgroundColor: LocalConfig.COLOR.BLACK,
                  padding: Dimensions.get('window').width / 30,
                }}>
                <View style={{ flex: 4 }}>
                  <Text
                    style={{
                      fontFamily: 'verdanab',
                      color: LocalConfig.COLOR.WHITE,
                      fontSize: 13,
                    }}>
                    {`${itemModelItem.item.menu_name}`.toUpperCase()}
                  </Text>
                  <Text
                    style={{
                      fontFamily: 'Proxima',
                      color: LocalConfig.COLOR.WHITE,
                      fontSize: 15,
                    }}>
                    {`\u20B9 ${itemModelItem.item.price}`}
                  </Text>
                  {itemModelItem.item.description != '' &&
                    (showMore.id != itemModelItem.item.id ? (
                      <Text
                        numberOfLines={2}
                        onPress={e => {
                          setShowMore({
                            id: itemModelItem.item.id,
                            description: itemModelItem.item.description,
                          });
                        }}
                        style={{
                          color: LocalConfig.COLOR.WHITE_LIGHT,
                          paddingVertical: 8,
                          fontFamily: 'Proxima Nova Font',
                          textAlign: 'justify',
                          paddingRight: 8,
                        }}>
                        {itemModelItem.item.description}
                      </Text>
                    ) : (
                      <Text
                        onPress={e => {
                          setShowMore({
                            id: null,
                            description: null,
                          });
                        }}
                        style={{
                          color: LocalConfig.COLOR.BLACK_LIGHT,
                          paddingVertical: 8,
                          fontFamily: 'Proxima Nova Font',
                          textAlign: 'justify',
                          paddingRight: 8,
                        }}>
                        {showMore.description}
                      </Text>
                    ))}
                </View>
                <View
                  style={{
                    flex: 1.5,
                    justifyContent: 'center',
                    alignItems: 'center',
                  }}>
                  <View
                    style={{
                      marginVertical: 10,
                      elevation: 10,
                      backgroundColor: itemModelItem.item.status == 0 ? LocalConfig.COLOR.UI_COLOR_LITE_TWICE : LocalConfig.COLOR.UI_COLOR,
                      width: '100%',
                      padding: 8,
                      borderRadius: 8,
                      justifyContent: 'center',
                    }}>
                    {itemModelItem.item.status == 0 ? (
                      itemModelItem.item.timing_state == 0 ? (
                        <Text
                          style={{
                            color: LocalConfig.COLOR.BLACK,
                            fontFamily: 'Proxima Nova Bold',
                            textAlign: 'center',
                            fontSize: 15,
                          }}>
                          SOLD
                        </Text>
                      ) : (
                        <Text
                          style={{
                            color: LocalConfig.COLOR.BLACK,
                            fontFamily: 'Proxima Nova Bold',
                            textAlign: 'center',
                          }}>{`Next available at ${itemModelItem.item.start}`}</Text>
                      )
                    ) : allMenuFromRedux[itemModelItem.catIndex]?.data[itemModelItem.itemIndex]?.qty <= 0 ? (
                      <TouchableOpacity
                        onPress={() => {
                          if (loadingItem != itemModelItem.item.id)
                            addingItem({
                              catIndex: itemModelItem.item.catIndex,
                              item: itemModelItem.item,
                              itemIndex: itemModelItem.item.itemIndex,
                              price: itemModelItem.item.price || itemModelItem.item.dprice,
                            });
                        }}
                        style={{
                          flexDirection: 'row',
                          justifyContent: 'center',
                          alignItems: 'center',
                          width: '100%',
                        }}>
                        <Text
                          style={{
                            color: LocalConfig.COLOR.BLACK,
                            fontFamily: 'Proxima Nova Bold',
                            fontSize: 15,
                          }}>
                          {loadingItem != itemModelItem.item.id ? (
                            `ADD`
                          ) : (
                            <ActivityIndicator
                              size={
                                Math.round(Dimensions.get('screen').width) / 20
                              }
                              color={LocalConfig.COLOR.BLACK}
                            />
                          )}
                        </Text>
                      </TouchableOpacity>
                    ) : (
                      <View
                        style={{
                          flexDirection: 'row',
                          justifyContent: 'space-around',
                          alignItems: 'center',
                        }}>
                        {loadingItem != itemModelItem.item.id && (
                          <MaterialIcons
                            name="remove"
                            size={20}
                            color={LocalConfig.COLOR.BLACK}
                            onPress={() => {
                              removingItem({
                                catIndex: itemModelItem.item.catIndex,
                                item: allMenuFromRedux[itemModelItem.catIndex]
                                  .data[itemModelItem.itemIndex],
                                itemIndex: itemModelItem.item.itemIndex,
                                price: itemModelItem.item.price,
                              });
                            }}
                          />
                        )}
                        <Text
                          style={{
                            color: LocalConfig.COLOR.BLACK,
                            fontFamily: 'Proxima Nova Bold',
                            fontSize: 18,
                            textAlign: 'center',
                          }}>
                          {loadingItem != itemModelItem.item.id ? (
                            allMenuFromRedux[itemModelItem.catIndex].data[
                              itemModelItem.itemIndex
                            ].qty
                          ) : (
                            <ActivityIndicator
                              size={
                                Math.round(Dimensions.get('screen').width) / 20
                              }
                              color={LocalConfig.COLOR.BLACK}
                            />
                          )}
                        </Text>
                        {loadingItem != itemModelItem.item.id && (
                          <MaterialIcons
                            name="add"
                            size={20}
                            color={LocalConfig.COLOR.BLACK}
                            onPress={() => {
                              addingItem({
                                catIndex: itemModelItem.item.catIndex,
                                item: itemModelItem.item,
                                itemIndex: itemModelItem.item.itemIndex,
                                price: itemModelItem.item.price || itemModelItem.item.dprice,
                              });
                            }}
                          />
                        )}
                      </View>
                    )}
                  </View>
                </View>
              </View>
            </View>
          </View>
        </Modal>
      )}
      {/* VARIENT MODAL */}
      {addonsModalItem.item != undefined && (
        <Modal
          visible={addonsModal}
          transparent={true}
          onBackdropPress={() => {
            setAddonsModalItem({
              item: undefined,
              catIndex: undefined,
              itemIndex: undefined,
              variant: undefined,
              ingredient: undefined,
              totel: undefined,
            });
            setLoadingItem(null);
            setAddonsModal(false);
            setAddonsSeleted({
              radioButton: { id: [], price: [] },
              CheckBoxes: { id: [], price: [] },
              ingrediant: { id: [], price: [] },
            });
          }}
          onRequestClose={() => {
            setAddonsModalItem({
              item: undefined,
              catIndex: undefined,
              itemIndex: undefined,
              variant: undefined,
              ingredient: undefined,
              totel: undefined,
            });
            setAddonsSeleted({
              radioButton: { id: [], price: [] },
              CheckBoxes: { id: [], price: [] },
              ingrediant: { id: [], price: [] },
            });
            setLoadingItem(null);
            setAddonsModal(false);
          }}
          animationType={'fade'}
          style={{
            justifyContent: 'flex-end',
            alignItems: 'center',
            flex: 1,
            width: '100%',
            margin: 0,
            backgroundColor: 'rgba(0,0,0,0.2)',
            height: Dimensions.get('window').height / 2,
          }}>
          <View style={{ flex: 1 }} />
          <View>
            <View
              style={{
                flexDirection: 'row',
                height: (Dimensions.get('window').height / 100) * 7,
                backgroundColor: LocalConfig.COLOR.UI_COLOR,
                alignItems: 'center',
                width: Dimensions.get('window').width,
                justifyContent: 'center',
                borderTopLeftRadius: 20,
                borderTopRightRadius: 20,
              }}>
              <View style={{ flex: 1 }} />
              <Text
                style={{
                  flex: 10,
                  textAlign: 'center',
                  fontFamily: 'verdanab',
                  color: LocalConfig.COLOR.BLACK,
                }}>
                {`${addonsModalItem.item.menu_name}`.toUpperCase()}
              </Text>
              <TouchableOpacity
                style={{ flex: 1 }}
                onPress={() => {
                  setAddonsModalItem({
                    item: undefined,
                    catIndex: undefined,
                    itemIndex: undefined,
                    variant: undefined,
                    ingredient: undefined,
                    totel: undefined,
                  });
                  setAddonsSeleted({
                    radioButton: { id: [], price: [] },
                    CheckBoxes: { id: [], price: [] },
                    ingrediant: { id: [], price: [] },
                  });
                  setLoadingItem(null);
                  setAddonsModal(false);
                }}>
                <MaterialIcons
                  name="close"
                  size={25}
                  color={LocalConfig.COLOR.BLACK}
                />
              </TouchableOpacity>
            </View>
          </View>
          <View
            style={{
              width: Dimensions.get('screen').width,
              backgroundColor: LocalConfig.COLOR.BLACK,
            }}>
            {addonsModalItem.item.wholecat == 3 && (
              <TextInput
                style={{
                  height: 40,
                  margin: 12,
                  borderWidth: 1,
                  padding: 10,
                  borderColor: LocalConfig.COLOR.UI_COLOR,
                  borderRadius: 5,
                  color: LocalConfig.COLOR.WHITE,
                }}
                onChangeText={qty => setWholeCat3Input(qty)}
                keyboardType={'number-pad'}
                autoFocus={true}
                value={
                  wholeCat3Input <= 0 || isNaN(wholeCat3Input)
                    ? undefined
                    : `${wholeCat3Input}`
                }
                maxLength={5}
                placeholderTextColor={LocalConfig.COLOR.BLACK_LIGHT}
                placeholder={'Quantity'}
              />
            )}
            {addonsModalItem.ingredient?.length > 0 && <Text
              style={{
                fontSize: 18,
                color: LocalConfig.COLOR.WHITE,
                padding: 10,
                fontFamily: 'verdanab',
              }}>
              {"Addons"}
            </Text>}
            <FlatList
              data={addonsModalItem.ingredient}
              renderItem={ingrediant => (
                <View
                  style={{
                    flexDirection: 'row',
                    alignItems: 'center',
                    justifyContent: 'space-between',
                    paddingVertical: 10,
                    paddingHorizontal: 20,
                  }}>
                  <Text
                    style={{
                      color: LocalConfig.COLOR.WHITE,
                      fontFamily: 'Proxima Nova Font',
                    }}>
                    {ingrediant.item.item_name}{' '}
                    {ingrediant.item.price != 0 &&
                      `\u20B9 ` + ingrediant.item.price}
                  </Text>
                  <CheckBox
                    value={addonsSeleted.ingrediant.id.includes(
                      ingrediant.item.id,
                    )}
                    onValueChange={ev => {
                      var totel = parseFloat(addonsModalItem.item.price);
                      if (!ev) {
                        var myidArray = addonsSeleted.ingrediant.id;
                        var mypriceArray = addonsSeleted.ingrediant.price;
                        var mynameArray = addonsSeleted.ingrediant.name;
                        const index = myidArray.indexOf(ingrediant.item.id);
                        if (index > -1) {
                          myidArray.splice(index, 1); // 2nd parameter means remove one item only
                          mypriceArray.splice(index, 1); // 2nd parameter means remove one item only
                          mynameArray.splice(index, 1); // 2nd parameter means remove one item only
                        }
                        mypriceArray.map(radioButtonPrice => {
                          if (radioButtonPrice != '')
                            totel += parseFloat(radioButtonPrice);
                        });
                        addonsSeleted.CheckBoxes.price.map(radioButtonPrice => {
                          if (radioButtonPrice != '')
                            totel += parseFloat(radioButtonPrice);
                        });
                        addonsSeleted.radioButton.price.map(
                          radioButtonPrice => {
                            if (radioButtonPrice != '')
                              totel += parseFloat(radioButtonPrice);
                          },
                        );
                        setAddonsModalItem({
                          catIndex: addonsModalItem.item.catIndex,
                          ingredient: addonsModalItem.ingredient,
                          item: addonsModalItem.item,
                          itemIndex: addonsModalItem.item.itemIndex,
                          variant: addonsModalItem.variant,
                          totel,
                        });
                        setAddonsSeleted({
                          radioButton: addonsSeleted.radioButton,
                          CheckBoxes: addonsSeleted.CheckBoxes,
                          ingrediant: {
                            id: myidArray,
                            price: mypriceArray,
                            name: mynameArray,
                          },
                        });
                      } else {
                        // ADD NEW
                        var [myidArray, mypriceArray, mynameArray] = [
                          addonsSeleted.ingrediant.id,
                          addonsSeleted.ingrediant.price,
                          addonsSeleted.ingrediant.name,
                        ];
                        mypriceArray.push(ingrediant.item.price);
                        myidArray.push(ingrediant.item.id);
                        mynameArray.push(ingrediant.item.item_name);
                        setAddonsSeleted({
                          radioButton: addonsSeleted.radioButton,
                          CheckBoxes: addonsSeleted.CheckBoxes,
                          ingrediant: {
                            id: myidArray,
                            price: mypriceArray,
                            name: mynameArray,
                          },
                        });
                        mypriceArray.map(radioButtonPrice => {
                          if (radioButtonPrice != '')
                            totel += parseFloat(radioButtonPrice);
                        });
                        addonsSeleted.CheckBoxes.price.map(radioButtonPrice => {
                          if (radioButtonPrice != '')
                            totel += parseFloat(radioButtonPrice);
                        });
                        addonsSeleted.radioButton.price.map(
                          radioButtonPrice => {
                            if (radioButtonPrice != '')
                              totel += parseFloat(radioButtonPrice);
                          },
                        );
                        setAddonsModalItem({
                          catIndex: addonsModalItem.item.catIndex,
                          ingredient: addonsModalItem.ingredient,
                          item: addonsModalItem.item,
                          itemIndex: addonsModalItem.item.itemIndex,
                          variant: addonsModalItem.variant,
                          totel,
                        });
                      }
                    }}
                    key={ingrediant.index}
                    tintColors={{
                      true: LocalConfig.COLOR.UI_COLOR,
                      false: LocalConfig.COLOR.UI_COLOR,
                    }}
                  />
                </View>
              )}
            />
            <FlatList
              data={addonsModalItem.variant}
              renderItem={variantItems =>
                variantItems.item.data.length > 0 && (
                  <View>
                    <Text
                      style={{
                        fontSize: 18,
                        color: LocalConfig.COLOR.WHITE,
                        padding: 10,
                        fontFamily: 'verdanab',
                      }}>
                      {variantItems.item.heading}
                    </Text>
                    {variantItems.item.status == 1 && (
                      <FlatList
                        data={variantItems.item.data}
                        renderItem={radioButton => (
                          <RadioButton.Group
                            onValueChange={e =>
                              variantItems.item.data.map(item => {
                                var totel = parseFloat(
                                  addonsModalItem.item.price,
                                );
                                var item = addonsSeleted.radioButton;
                                var itemIndex = item.id.indexOf(e);
                                if (itemIndex == -1) {
                                  // new value
                                  item.id[variantItems.index] =
                                    variantItems.item.data[
                                      radioButton.index
                                    ].id;
                                  item.price[variantItems.index] =
                                    variantItems.item.data[
                                      radioButton.index
                                    ].price;
                                  item.name[variantItems.index] =
                                    variantItems.item.data[
                                      radioButton.index
                                    ].variant_name;
                                  setAddonsSeleted({
                                    CheckBoxes: addonsSeleted.CheckBoxes,
                                    radioButton: item,
                                    ingrediant: addonsSeleted.ingrediant,
                                  });
                                }
                                addonsSeleted.radioButton.price.map(
                                  radioButtonPrice => {
                                    if (radioButtonPrice != '')
                                      totel += parseFloat(radioButtonPrice);
                                  },
                                );
                                addonsSeleted.ingrediant.price.map(
                                  radioButtonPrice => {
                                    if (radioButtonPrice != '')
                                      totel += parseFloat(radioButtonPrice);
                                  },
                                );
                                addonsSeleted.CheckBoxes.price.map(
                                  radioButtonPrice => {
                                    if (radioButtonPrice != '')
                                      totel += parseFloat(radioButtonPrice);
                                  },
                                );
                                setAddonsModalItem({
                                  catIndex: addonsModalItem.item.catIndex,
                                  ingredient: addonsModalItem.ingredient,
                                  item: addonsModalItem.item,
                                  itemIndex: addonsModalItem.item.itemIndex,
                                  variant: addonsModalItem.variant,
                                  totel,
                                });
                              })
                            }
                            value={
                              addonsSeleted.radioButton.id[variantItems.index]
                            }>
                            <RadioButton.Item
                              labelStyle={{
                                fontSize: 14.5,
                                fontFamily: 'Proxima Nova Font',
                                color: LocalConfig.COLOR.WHITE,
                              }}
                              // {ingrediant.item.price != 0 && `\u20B9 ` + ingrediant.item.price}
                              label={`${radioButton.item.variant_name} ${radioButton.item.price != 0
                                ? `\u20B9 ` + radioButton.item.price
                                : ''
                                }`}
                              mode={'android'}
                              value={radioButton.item.id}
                              uncheckedColor={LocalConfig.COLOR.UI_COLOR}
                              color={LocalConfig.COLOR.UI_COLOR}
                            />
                          </RadioButton.Group>
                        )}
                      />
                    )}
                    {variantItems.item.status == 0 && (
                      <FlatList
                        data={variantItems.item.data}
                        key={({ item, index }) => index}
                        renderItem={radioButton => (
                          <View
                            style={{
                              flexDirection: 'row',
                              alignItems: 'center',
                              justifyContent: 'space-between',
                              paddingVertical: 10,
                              paddingHorizontal: 20,
                            }}>
                            <Text
                              style={{
                                color: LocalConfig.COLOR.WHITE,
                                fontFamily: 'Proxima Nova Font',
                              }}>
                              {`${radioButton.item.variant_name} ${radioButton.item.price != 0
                                ? `\u20B9 ` + radioButton.item.price
                                : ''
                                }`}
                            </Text>
                            <CheckBox
                              value={addonsSeleted.CheckBoxes.id.includes(
                                radioButton.item.id,
                              )}
                              onValueChange={ev => {
                                var totel = parseFloat(
                                  addonsModalItem.item.price,
                                );
                                if (!ev) {
                                  var myidArray = addonsSeleted.CheckBoxes.id;
                                  var mypriceArray =
                                    addonsSeleted.CheckBoxes.price;
                                  var mynameArray =
                                    addonsSeleted.CheckBoxes.name;
                                  const index = myidArray.indexOf(
                                    radioButton.item.id,
                                  );
                                  if (index > -1) {
                                    myidArray.splice(index, 1); // 2nd parameter means remove one item only
                                    mypriceArray.splice(index, 1); // 2nd parameter means remove one item only
                                    mynameArray.splice(index, 1); // 2nd parameter means remove one item only
                                  }
                                  setAddonsSeleted({
                                    radioButton: addonsSeleted.radioButton,
                                    CheckBoxes: {
                                      id: myidArray,
                                      price: mypriceArray,
                                      name: mynameArray,
                                    },
                                    ingrediant: addonsSeleted.ingrediant,
                                  });
                                  mypriceArray.map(CheckBoxesPrice => {
                                    if (CheckBoxesPrice != '')
                                      totel += parseFloat(CheckBoxesPrice);
                                  });
                                  addonsSeleted.ingrediant.price.map(
                                    radioButtonPrice => {
                                      if (radioButtonPrice != '')
                                        totel += parseFloat(radioButtonPrice);
                                    },
                                  );
                                  addonsSeleted.radioButton.price.map(
                                    radioButtonPrice => {
                                      if (radioButtonPrice != '')
                                        totel += parseFloat(radioButtonPrice);
                                    },
                                  );
                                  setAddonsModalItem({
                                    catIndex: addonsModalItem.item.catIndex,
                                    ingredient: addonsModalItem.ingredient,
                                    item: addonsModalItem.item,
                                    itemIndex: addonsModalItem.item.itemIndex,
                                    variant: addonsModalItem.variant,
                                    totel,
                                  });
                                } else {
                                  // ADD NEW
                                  var [myidArray, mypriceArray, mynameArray] = [
                                    addonsSeleted.CheckBoxes.id,
                                    addonsSeleted.CheckBoxes.price,
                                    addonsSeleted.CheckBoxes.name,
                                  ];
                                  mypriceArray.push(radioButton.item.price);
                                  myidArray.push(radioButton.item.id);
                                  mynameArray.push(
                                    radioButton.item.variant_name,
                                  );
                                  setAddonsSeleted({
                                    radioButton: addonsSeleted.radioButton,
                                    CheckBoxes: {
                                      id: myidArray,
                                      price: mypriceArray,
                                      name: mynameArray,
                                    },
                                    ingrediant: addonsSeleted.ingrediant,
                                  });
                                  mypriceArray.map(radioButtonPrice => {
                                    if (radioButtonPrice != '')
                                      totel += parseFloat(radioButtonPrice);
                                  });
                                  addonsSeleted.ingrediant.price.map(
                                    radioButtonPrice => {
                                      if (radioButtonPrice != '')
                                        totel += parseFloat(radioButtonPrice);
                                    },
                                  );
                                  addonsSeleted.radioButton.price.map(
                                    radioButtonPrice => {
                                      if (radioButtonPrice != '')
                                        totel += parseFloat(radioButtonPrice);
                                    },
                                  );
                                  setAddonsModalItem({
                                    catIndex: addonsModalItem.item.catIndex,
                                    ingredient: addonsModalItem.ingredient,
                                    item: addonsModalItem.item,
                                    itemIndex: addonsModalItem.item.itemIndex,
                                    variant: addonsModalItem.variant,
                                    totel,
                                  });
                                }
                              }}
                              key={radioButton.index}
                              tintColors={{
                                true: LocalConfig.COLOR.UI_COLOR,
                                false: LocalConfig.COLOR.UI_COLOR,
                              }}
                            />
                          </View>
                        )}
                      />
                    )}
                  </View>
                )
              }
            />
          </View>
          <TouchableOpacity
            onPress={() => {
              let qty;
              if (addonsModalItem.item.wholecat == 2) qty = 0.25;
              else if (addonsModalItem.item.wholecat == 3)
                qty = wholeCat3Input;
              else
                qty = 1;
              addaddonsItem({
                catIndex: addonsModalItem.item.catIndex,
                item: addonsModalItem.item,
                itemIndex: addonsModalItem.item.itemIndex,
                addonsSeleted: addonsSeleted,
                qty: qty,
                price: addonsModalItem.totel,
              });
            }}
            style={{
              backgroundColor: LocalConfig.COLOR.UI_COLOR,
              height: (Dimensions.get('window').height / 100) * 7,
              display: 'flex',
              width: '100%',
              flexDirection: 'row',
              alignItems: 'center',
              justifyContent: 'space-around',
            }}>
            <Text
              style={{
                fontFamily: 'verdanab',
                color: LocalConfig.COLOR.BLACK,
              }}>
              TOTEL | {`\u20B9 ${addonsModalItem.totel}`}
            </Text>
            <View />
            <Text
              style={{
                fontFamily: 'verdanab',
                color: LocalConfig.COLOR.BLACK,
              }}>
              ADD ITEM
            </Text>
          </TouchableOpacity>
        </Modal>
      )}
      {/* REPEAT MODAL */}
      {repeatModalItem.item && (
        <Modal
          visible={repeatModal}
          onBackdropPress={() => {
            setRepeatModal(false);
            setRepeatModalItem({
              item: undefined,
              catIndex: undefined,
              itemIndex: undefined,
              price: undefined,
            });
          }}
          onRequestClose={() => {
            setRepeatModal(false);
            setRepeatModalItem({
              item: undefined,
              catIndex: undefined,
              itemIndex: undefined,
              price: undefined,
            });
          }}
          style={{
            justifyContent: 'flex-end',
            flex: 1,
            width: '100%',
            margin: 0,
            backgroundColor: 'rgba(0,0,0,0.5)',
          }}
          animationType="slide"
          transparent={true}>
          <View style={{ flex: 1 }} />
          <View>
            <View
              style={{
                backgroundColor: LocalConfig.COLOR.UI_COLOR,
                flexDirection: 'row',
                alignItems: 'center',
                width: '100%',
                padding: 15,
                borderTopLeftRadius: 20,
                borderTopRightRadius: 20,
              }}>
              <Text
                style={{
                  flex: 1,
                  textAlign: 'center',
                  //fontWeight: 'bold',
                  letterSpacing: 2,
                  color: LocalConfig.COLOR.BLACK,
                  fontFamily: 'verdanab',
                }}>
                {`${repeatModalItem.item.menu_name}`.toUpperCase()}
              </Text>
              <TouchableOpacity onPress={() => setRepeatModal(false)}>
                <View style={{}}>
                  <MaterialIcons
                    name="close"
                    size={25}
                    color={LocalConfig.COLOR.BLACK}
                  />
                </View>
              </TouchableOpacity>
            </View>
            <View
              style={{
                flexDirection: 'row',
                backgroundColor: LocalConfig.COLOR.BLACK,
                paddingVertical: 30,
              }}>
              <TouchableOpacity
                style={{
                  flex: 1,
                  backgroundColor: LocalConfig.COLOR.BLACK,
                  justifyContent: 'center',
                  alignItems: 'center',
                  padding: 10,
                  margin: 10,
                  borderColor: LocalConfig.COLOR.UI_COLOR,
                  borderWidth: 2,
                  borderRadius: 10,
                }}
                onPress={() => {
                  addingItem({
                    catIndex: repeatModalItem.item.catIndex,
                    item: repeatModalItem.item,
                    itemIndex: repeatModalItem.item.itemIndex,
                    price: repeatModalItem.price || repeatModalItem.dprice,
                    repeat: true,
                  });
                  setRepeatModal(false);
                  setRepeatModalItem({
                    item: undefined,
                    catIndex: undefined,
                    itemIndex: undefined,
                    price: undefined,
                  });
                }}>
                <Text
                  style={{
                    fontFamily: 'verdanab',
                    color: LocalConfig.COLOR.UI_COLOR,
                  }}>
                  I'LL CHOOSE
                </Text>
              </TouchableOpacity>
              <TouchableOpacity
                style={{
                  flex: 1,
                  backgroundColor: LocalConfig.COLOR.UI_COLOR,
                  justifyContent: 'center',
                  alignItems: 'center',
                  padding: 10,
                  margin: 10,
                  borderRadius: 10,
                }}
                onPress={() => {
                  var qty;
                  if (
                    // repeatModalItem.item.wholecat == 3 ||
                    repeatModalItem.item.wholecat == 5 ||
                    repeatModalItem.item.wholecat == 1
                  ) {
                    qty = parseFloat(repeatModalItem.item.qty) + 1;
                  } else {
                    var qty;
                    qty = parseFloat(repeatModalItem.item.qty) + 0.25;
                  }
                  dispatch(
                    addToCart([
                      repeatModalItem.item.catIndex,
                      repeatModalItem.item.itemIndex,
                      qty,
                      repeatModalItem.price,
                      repeatModalItem.item,
                      'update',
                    ]),
                  );
                  setRepeatModal(false);
                  setRepeatModalItem({
                    item: undefined,
                    catIndex: undefined,
                    itemIndex: undefined,
                    price: undefined,
                  });
                  refreshME();
                }}>
                <Text
                  style={{
                    fontFamily: 'verdanab',
                    color: LocalConfig.COLOR.BLACK,
                  }}>
                  REPEAT
                </Text>
              </TouchableOpacity>
            </View>
          </View>
        </Modal>
      )}
      <Modal
        visible={couponInfoModal}
        onBackdropPress={() => openCouponInfoModal({})}
        onRequestClose={() => openCouponInfoModal({})}
        style={{
          justifyContent: 'center',
          flex: 1,
          margin: 10,
          backgroundColor: 'rgba(0,0,0,0.5)',
          alignItems: 'center',
        }}
        animationType="slide"
        transparent={true}
      >
        <View style={{ flex: 1 }} />
        <View
          style={{
            backgroundColor: LocalConfig.COLOR.UI_COLOR,
            flexDirection: 'row',
            alignItems: 'center',
            width: '100%',
            padding: 10,
            borderTopLeftRadius: 20,
            borderTopRightRadius: 20,
          }}>
          <TouchableOpacity onPress={() => openCouponInfoModal({})}>
            <View style={{}}>
              <MaterialIcons
                name="close"
                size={25}
                color={LocalConfig.COLOR.BLACK}
              />
            </View>
          </TouchableOpacity>
        </View>
        <View style={{
          paddingVertical: 20, paddingHorizontal: 10, backgroundColor: LocalConfig.COLOR.BLACK, width: '100%', borderColor: LocalConfig.COLOR.UI_COLOR,
          borderLeftWidth: 2,
          borderBottomWidth: 2,
          borderRightWidth: 2,
          borderBottomLeftRadius: 10,
          borderBottomRightRadius: 10
        }}>
          <Text
            style={{
              color: LocalConfig.COLOR.WHITE,
              fontSize: 14,
              backgroundColor: LocalConfig.COLOR.UI_COLOR_LITE,
              opacity: 0.5,
              marginHorizontal: 3,
              paddingHorizontal: 10,
              borderLeftWidth: 3,
              borderColor: LocalConfig.COLOR.UI_COLOR,
              maxWidth: '30%',
              padding: 5
            }}>
            Upto {couponInfoItem.coupon?.max_free_prdt_qty}
          </Text>
          <Text style={{ color: LocalConfig.COLOR.WHITE, fontFamily: 'verdanab', textAlign: 'justify', fontSize: 15 }}>Buy {couponInfoItem.coupon?.max_main_prdt_qty} products and get up to {couponInfoItem.coupon?.free_prdct_qty} {couponInfoItem.coupon?.free_prdct != couponInfoItem.id && `${couponInfoItem.coupon?.free_prdct_menu_name} `}for free! When you add {couponInfoItem.coupon?.max_main_prdt_qty} or more products to your cart, we'll include {couponInfoItem.coupon?.free_prdct_qty}{couponInfoItem.coupon?.free_prdct != couponInfoItem.id && ` ${couponInfoItem.coupon?.free_prdct_menu_name} `}, and its cost will be discounted at checkout until a maximum of {couponInfoItem.coupon?.max_free_prdt_qty} free products have been applied. Plus, with no extra cost, as 1 item price {`\u20B9${couponInfoItem.price}`}. This promotion is valid for a limited time only, so start shopping now to take advantage of this great offer!</Text>
        </View>
      </Modal>
      <Modal
        visible={meatInputModal}
        onBackdropPress={() => setMeatInputModal(false)}
        onRequestClose={() => setMeatInputModal(false)}
        style={{
          justifyContent: 'center',
          flex: 1,
          margin: 0,
          backgroundColor: 'rgba(0,0,0,0.5)',
          alignItems: 'center',
        }}
        animationType="slide"
        transparent={true}
      >
        <BlurView
          style={styles.absolute}
          blurType="dark"
          blurAmount={10}
          reducedTransparencyFallbackColor="white"
        />
        <View style={{ flex: 1 }} />
        <View
          style={{
            backgroundColor: LocalConfig.COLOR.UI_COLOR,
            padding: 10,
            borderTopLeftRadius: 20,
            borderTopRightRadius: 20,
          }}
        >
          <View style={{
            flexDirection: 'row',
            justifyContent: 'space-between',
            alignItems: 'center',
          }}>
            <Text style={{ padding: 5, textAlign: 'left', fontFamily: 'verdanab', color: LocalConfig.COLOR.BLACK }}>{meatInputModalItem.item.menu_name}</Text>
            <TouchableNativeFeedback onPress={() => setMeatInputModal(false)}><MaterialIcons name='close' color={LocalConfig.COLOR.BLACK} /></TouchableNativeFeedback>
          </View>
          <Image
            source={{
              uri: menu_item_icon_IMAGE + meatInputModalItem.item.menu_image,
            }}
            style={{
              width: Dimensions.get('window').width / 2,
              height: Dimensions.get('window').width / 2,
              borderRadius: 20,
              alignSelf: 'center'
            }}
            resizeMode={'stretch'}
          />
          <TextInput
            placeholder={'Enter Your Quantity'}
            placeholderTextColor={LocalConfig.COLOR.BLACK}
            onChangeText={qty => setMeatInput(qty)}
            style={{ color: LocalConfig.COLOR.BLACK, alignSelf: 'center' }}
            keyboardType={'number-pad'}
            value={meatInput > 0 ? meatInput : undefined}
          />
          <TouchableOpacity
            disabled={(meatInput * parseFloat(meatInputModalItem.item.dprice)) <= 0}
            style={{
              padding: 10,
              borderRadius: 20,
              backgroundColor: (meatInput * parseFloat(meatInputModalItem.item.dprice)) <= 0 ? LocalConfig.COLOR.BLACK_LIGHT_LITER : LocalConfig.COLOR.BLACK,
              width: Dimensions.get('window').width / 2,
              alignSelf: 'center'
            }}
            onPress={() =>

              // console.log({
              //   catIndex: meatInputModalItem.catIndex,
              //   item: "meatInputModalItem.item",
              //   itemIndex: meatInputModalItem.itemIndex,
              //   price: (meatInput * parseFloat(meatInputModalItem.item.dprice))
              // })
              addingItem({
                catIndex: meatInputModalItem.item.catIndex,
                item: meatInputModalItem.item,
                itemIndex: meatInputModalItem.item.itemIndex,
                price: parseFloat(meatInputModalItem.item.dprice)
              })
            }
          ><Text style={{ color: LocalConfig.COLOR.UI_COLOR, textAlign: 'center', fontWeight: 'bold' }}>ADD{meatInput > 0 && ` \u20B9${meatInput * parseFloat(meatInputModalItem.item.dprice)}`}</Text></TouchableOpacity>
        </View>
      </Modal>
    </View>
  )
}

export default SearchScreen

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: LocalConfig.COLOR.BLACK,
  },
  input: {
    backgroundColor: LocalConfig.COLOR.BLACK_LIGHT_LITER,
    borderTopLeftRadius: 10,
    borderBottomLeftRadius: 10,
    flex: 1
  },
  clearSearch: {
    backgroundColor: LocalConfig.COLOR.BLACK_LIGHT_LITER,
    borderTopRightRadius: 10,
    borderBottomRightRadius: 10,
    justifyContent: 'center',
    alignItems: 'center',
    flex: 0.2
  },
  search_items_view: {
    paddingHorizontal: 10
  }
})
