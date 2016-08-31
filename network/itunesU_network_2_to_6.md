## 2강

* 특정 프로세스를 찾기 위해서는 특정 IP(호스트) + port(프로세스)가 필요함 -> 특정 소켓을 지칭할 수 있음
* 애플리케이션 계층(HTTP 등) > 트랜스포트 계층(TCP, UDP)
* TCP : 신뢰성 있는 통신, 데이터 유실 없음, 보안 없음, 타임에 대한 개런티 없음
* UDP : 비신뢰성 통신, 데이터 유실 있음, 보안 없음, 타임에 대한 개런티 없음
* 통신하려는 상대와 같은 프로토콜로 통신해야함(TCP - TCP)
* HTTP는 어플리케이션 계층이기 때문에 멀리 떨어진 서버와 신뢰성 있는 통신을 하기 위해서는 트랜스포트 계층의 TCP 프로토콜로 먼저 서버와 소켓이 연결이 되어야한다.
* non-persistent HTTP(매 request마다 tcp 새로 연결) VS persistent HTTP(현재 버전의 default, 한번 연결된 TCP 프로토콜은 끊지 않고 지속함)
import time
from datetime import datetime

from django.core.files.images import get_image_dimensions
from django.core.urlresolvers import reverse
from django.db import transaction
from django.db.models import F
from django.http import HttpResponseRedirect
from django.utils import timezone
from django.views.generic.edit import CreateView, UpdateView
from django.views.generic.list import ListView

from exid.auth import permissions
from exid.custom_view.PermissionMixin import PermissionMixin
from exid.models.TCategory import TCategory
from exid.models.TV4Banner import TV4Banner
from exid.models.TV4Section import TV4Section
from exid.models.TV4SectionItem import TV4SectionItem
from exid.util import kage_util, app_util

BANNER_IMG_SIZE = {
    # SIZE_KEY: (배너명, width, height), # Section type
    'DEFAULT': ("일반 배너", 720, 300),  # BANNER, BIGBANNER
    'HOME_TOP_BIG': ("홈 상단 빅배너", 720, 454),  # TOP_BIG & position=0
    'TOP_BIG': ("상단 빅배너", 720, 360),  # TOP_BIG, EVENT
    'LINE': ("띠 배너", 720, 140,),  # LINEBANNER
    'TODAYGIFT': ("오늘의 선물 배너", 720, 270),  # TGBANNER
}


def get_image_size_key(position, section_type):
    size_key = 'DEFAULT'
    if section_type == 'TOPBIG':
        size_key = 'TOP_BIG'
        if position == "0":
            size_key = 'HOME_TOP_BIG'
    elif section_type == 'EVENT':
        size_key = 'TOP_BIG'
    elif section_type == 'LINEBANNER':
        size_key = 'LINE'
    elif section_type == 'TGBANNER':
        size_key = 'TODAYGIFT'
    return size_key


def is_valid_image(img_file, err_container, size_key):
    w, h = get_image_dimensions(img_file)
    if w != BANNER_IMG_SIZE[size_key][1] or h != BANNER_IMG_SIZE[size_key][2]:
        err_container.append({"msg": "이미지 사이즈가 맞지 않습니다. %s 의 이미지 사이즈는 %s x %s 입니다 " % BANNER_IMG_SIZE[size_key]})
        return False
    return True


class BannerListView(PermissionMixin, ListView):
    permission_required = permissions.ManageBanner()
    template_name = "hani/banner/list.html"
    model = TV4SectionItem
    paginate_by = 10

    request = None
    # 검색필터의 Default 값
    search = {"os": 'C', "date_type": 'on_air', "activation": 'A', 'page': 1}
    url_params = dict()  # 지속적으로 Tracking 해야하는 값들을 저장

    def get(self, request, *args, **kwargs):
        self.url_params = kwargs
        self.request = request
        return super(BannerListView, self).get(request, *args, **kwargs)

    def get_queryset(self):

        # Section 으로 먼저 필터를 먹인다.
        section = TV4Section.objects.get(uid=self.url_params['section_uid'])
        object_list = self.model.objects.filter(section=section)
        filter_info = app_util.make_search_context(self.request, self.search)

        # Section item 에 대한 필터
        os_type_common = 'C'
        if filter_info['date_type'] == 'on_air':
            current_date = timezone.make_aware(datetime.now())
            object_list = object_list.filter(start_dt__lte=current_date, end_dt__gte=current_date)
        elif filter_info['date_type'] == 'waiting_for_publish':
            current_date = timezone.make_aware(datetime.now())
            object_list = object_list.filter(start_dt__gt=current_date)

        if filter_info['os'] != os_type_common:
            object_list = object_list.filter(os__in=(filter_info['os'], os_type_common))

        if filter_info['activation'] != 'A':
            object_list = object_list.filter(activation=filter_info['activation'])

        object_list = object_list.prefetch_related('content_object').order_by('item_order')
        return object_list

    def get_context_data(self, **kwargs):
        context = super(BannerListView, self).get_context_data(**kwargs)
        context['params'] = self.url_params
        context['search'] = app_util.make_search_context(self.request, self.search)
        context['activation_choices'] = TV4SectionItem.ACTIVATIONS
        context['os_choices'] = TV4SectionItem.OS_TYPES
        context['os_choices_dic'] = app_util.get_dict_by_choices(TV4SectionItem.OS_TYPES)
        context['date_type_choices'] = TV4SectionItem.DATE_TYPES
        return context


class BannerUpdateView(PermissionMixin, UpdateView):
    permission_required = permissions.ManageBanner()
    model = TV4Banner
    fields = ['title', 'bgcolor', 'destination_type']
    template_name = "hani/banner/update.html"
    request = None
    url_params = dict()
    errors = []

    def get(self, request, *args, **kwargs):
        self.url_params = kwargs
        self.request = request
        return super(BannerUpdateView, self).get(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        self.errors = []

        section = TV4Section.objects.get(uid=kwargs['section_uid'])
        if 'img_data' in request.FILES and request.FILES['img_data']:
            img_size_key = get_image_size_key(kwargs['position'], section.section_type)
            if is_valid_image(request.FILES['img_data'], self.errors, img_size_key):
                request.POST['img_url'] = kage_util.upload_to_kage(request.FILES['img_data'])

        if len(self.errors) == 0:
            # Banner Data
            banner = TV4Banner.objects.get(uid=kwargs['pk'])
            banner_essentials = ["title", "destination_type", "destination"]
            made_banner = app_util.make_model_obj(banner, request.POST, banner_essentials, ["bgcolor", "img_url"])

            if made_banner['success']:
                banner.save()

                # Section item Data
                section_item = TV4SectionItem.objects.get(object_id=banner.uid, content_type_id=10)
                section_item_fields = ["item_order", "os", "activation", "start_dt", "end_dt"]
                made_section_item = app_util.make_model_obj(section_item, request.POST, section_item_fields)

                if made_section_item['success']:
                    section_item.save()
                else:
                    self.errors.append({'msg': made_section_item['message']})
            else:
                self.errors.append({'msg': made_banner['message']})

        return self.get(request, *args, **kwargs)

    def get_context_data(self, **kwargs):
        context = super(BannerUpdateView, self).get_context_data(**kwargs)
        context['errors'] = self.errors
        context['params'] = self.url_params
        section = TV4Section.objects.get(pk=self.url_params['section_uid'])
        context['img_size'] = BANNER_IMG_SIZE[get_image_size_key(self.url_params['position'], section.section_type)]

        context['os_choices'] = TV4SectionItem.OS_TYPES
        context['activation_choices'] = TV4SectionItem.ACTIVATIONS
        context['category_list'] = TCategory.objects.filter(parent_uid=0).exclude(uid=0).all()
        context['section_item'] = TV4SectionItem.objects.get(object_id=context['object'].uid, content_type_id=10)

        if hasattr(self.request, 'POST'):
            obj_fields = ['destination_type', 'destination']
            sitem_fields = ["item_order", "os", "activation", "start_dt", "end_dt"]
            app_util.add_posted_data_to_context(context, self.request.POST, obj_fields)
            app_util.add_posted_data_to_context(context, self.request.POST, sitem_fields, target_context_key="section_item")
        if context['object'].img_url:
            context['object'].img_url = kage_util.get_kage_full_url(self.request, context['object'].img_url)
        return context


class BannerCreateView(PermissionMixin, CreateView):
    permission_required = permissions.ManageBanner()
    template_name = "hani/banner/create.html"
    model = TV4Banner
    fields = ['title', 'bgcolor', 'destination_type']
    url_params = dict()  # position, section_uid
    request = None
    errors = []

    def get(self, request, *args, **kwargs):
        self.url_params = kwargs
        self.request = request
        return super(BannerCreateView, self).get(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        self.errors = []

        section = TV4Section.objects.get(uid=kwargs['section_uid'])
        if 'img_data' in request.FILES and request.FILES['img_data']:
            img_size_key = get_image_size_key(kwargs['position'], section.section_type)
            if is_valid_image(request.FILES['img_data'], self.errors, img_size_key):
                request.POST['img_url'] = kage_util.upload_to_kage(request.FILES['img_data'])

        if len(self.errors) == 0:
            # Banner Data
            banner = TV4Banner()
            banner.banner_type = 'BA02'  # 일반 배너
            banner_essentials = ["title", "destination_type", "destination", "img_url"]
            made_banner = app_util.make_model_obj(banner, request.POST, banner_essentials, ["bgcolor"])
            if made_banner['success']:
                banner.save()
                kwargs['pk'] = banner.uid

                # Section item Data
                section_item = TV4SectionItem()
                section_item.section = TV4Section.objects.get(uid=kwargs['section_uid'])
                section_item.object_id = banner.uid
                section_item.content_type_id = 10  # tv4banner
                section_item_fields = ["item_order", "os", "activation", "start_dt", "end_dt"]
                made_section_item = app_util.make_model_obj(section_item, request.POST, section_item_fields)

                if made_section_item['success']:
                    section_item.save()
                else:
                    banner.delete()
                    self.errors.append({'msg': made_section_item['message']})
            else:
                self.errors.append({'msg': made_banner['message']})

        if len(self.errors) == 0:
            return HttpResponseRedirect(reverse('banner_detail', kwargs=kwargs))
        else:
            return self.get(request, *args, **kwargs)

    def get_context_data(self, **kwargs):
        context = super(BannerCreateView, self).get_context_data(**kwargs)
        context['errors'] = self.errors
        context['params'] = self.url_params
        section = TV4Section.objects.get(pk=self.url_params['section_uid'])
        context['img_size'] = BANNER_IMG_SIZE[get_image_size_key(self.url_params['position'], section.section_type)]
        context['os_choices'] = TV4SectionItem.OS_TYPES
        context['activation_choices'] = TV4SectionItem.ACTIVATIONS
        context['category_list'] = TCategory.objects.filter(parent_uid=0).exclude(uid=0).all()

        if hasattr(self.request, 'POST'):
            obj_fields = ['destination_type', 'destination']
            sitem_fields = ["item_order", "os", "activation", "start_dt", "end_dt"]
            app_util.add_posted_data_to_context(context, self.request.POST, obj_fields)
            app_util.add_posted_data_to_context(context, self.request.POST, sitem_fields, target_context_key="section_item")
        return context


class WeeklyBannerListView(PermissionMixin, ListView):
    permission_required = permissions.ManageBanner()
    template_name = "hani/banner/weekly_list.html"
    model = TV4SectionItem
    paginate_by = 10

    request = None
    # 검색필터의 Default 값
    search = {"os": 'C', "date_type": 'on_air', "activation": 'A', "day": '0', 'page': 1}
    url_params = dict()  # 지속적으로 Tracking 해야하는 값들을 저장

    def get(self, request, *args, **kwargs):
        self.url_params = kwargs
        self.request = request
        return super(WeeklyBannerListView, self).get(request, *args, **kwargs)

    def get_queryset(self):

        # Section 으로 먼저 필터를 먹인다.
        section = TV4Section.objects.get(uid=self.url_params['section_uid'])
        object_list = self.model.objects.filter(section=section)
        filter_info = app_util.make_search_context(self.request, self.search)
        if not filter_info['day'] or filter_info['day'] == '0':
            filter_info['day'] = str(time.localtime().tm_wday + 1)

        # Section item 에 대한 필터
        os_type_common = 'C'
        if filter_info['date_type'] == 'on_air':
            current_date = timezone.make_aware(datetime.now())
            object_list = object_list.filter(start_dt__lte=current_date, end_dt__gte=current_date)
        elif filter_info['date_type'] == 'waiting_for_publish':
            current_date = timezone.make_aware(datetime.now())
            object_list = object_list.filter(start_dt__gt=current_date)

        if filter_info['os'] != os_type_common:
            object_list = object_list.filter(os__in=(filter_info['os'], os_type_common))

        if filter_info['activation'] != 'A':
            object_list = object_list.filter(activation=filter_info['activation'])

        object_list = object_list.filter(day=filter_info['day'])
        object_list = object_list.prefetch_related('content_object').order_by('item_order')
        return object_list

    def get_context_data(self, **kwargs):
        search_dic = app_util.make_search_context(self.request, self.search)
        self.kwargs['page'] = self.get_page(search_dic)
        context = super(WeeklyBannerListView, self).get_context_data(**kwargs)
        context['search'] = search_dic
        if not context['search']['day'] or context['search']['day'] == '0':
            context['search']['day'] = str(time.localtime().tm_wday + 1)
        context['params'] = self.url_params
        context['activation_choices'] = TV4SectionItem.ACTIVATIONS
        context['os_choices'] = TV4SectionItem.OS_TYPES
        context['os_choices_dic'] = app_util.get_dict_by_choices(TV4SectionItem.OS_TYPES)
        context['date_type_choices'] = TV4SectionItem.DATE_TYPES
        context['day_choices'] = TV4SectionItem.DAYS
        return context

    def get_page(self, search_dic: dict):
        """
        현재 페이지에 1개의 아이템이 남았을때 삭제하는 경우, 이전 페이지로 갈 수 있도록 함
        URL에 page가 기존 페이지로 유지되는 문제가 있음
        TODO: deleteview를 이용하든, 다른 방법을 찾아야함.
        :param search_dic
        :return:
        """
        if len(self.object_list) / self.paginate_by <= int(search_dic['page']) - 1:
            return int(search_dic['page']) - 1
        else:
            return search_dic['page']


class WeeklyBannerUpdateView(PermissionMixin, UpdateView):
    permission_required = permissions.ManageBanner()
    model = TV4Banner
    fields = ['title', 'bgcolor', 'destination_type']
    template_name = "hani/banner/weekly_update.html"
    url_params = dict()
    request = None
    errors = []

    def get(self, request, *args, **kwargs):
        self.request = request
        self.url_params = kwargs
        return super(WeeklyBannerUpdateView, self).get(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        # 에러 초기화
        self.errors = []
        banner = TV4Banner.objects.get(uid=kwargs['pk'])
        data = request.POST

        if 'img_data' in request.FILES and request.FILES['img_data']:
            section = TV4Section.objects.get(pk=kwargs['section_uid'])
            img_size_key = get_image_size_key(kwargs['position'], section.section_type)
            if is_valid_image(request.FILES['img_data'], self.errors, img_size_key):
                request.POST['img_url'] = kage_util.upload_to_kage(request.FILES['img_data'])

        if len(self.errors) == 0:
            keys = ["img_url", "destination_type", "destination", "bgcolor", "title"]
            for key in keys:
                if key in data:
                    setattr(banner, key, data[key])
            banner.save()

        return self.get(request, *args, **kwargs)

    def get_context_data(self, **kwargs):
        context = super(WeeklyBannerUpdateView, self).get_context_data(**kwargs)
        context['os_choices'] = TV4SectionItem.OS_TYPES
        context['activation_choices'] = TV4SectionItem.ACTIVATIONS
        context['day_choices'] = TV4SectionItem.DAYS
        context['errors'] = self.errors
        context['params'] = self.url_params
        req_data = app_util.collect_request_data(self.request, ['POST', 'GET'], ['day'])
        if 'day' in req_data:
            context['params']['day'] = req_data['day']
        context['section_items'] = TV4SectionItem.objects.filter(object_id=context['object'].uid, content_type_id=10).order_by('day')
        context['category_list'] = TCategory.objects.filter(parent_uid=0).exclude(uid=0).all()
        section = TV4Section.objects.get(pk=self.url_params['section_uid'])
        context['img_size'] = BANNER_IMG_SIZE[get_image_size_key(self.url_params['position'], section.section_type)]

        if hasattr(self.request, 'POST'):
            recover_fields = ['destination_type', 'destination']
            app_util.add_posted_data_to_context(context, self.request.POST, recover_fields)
        if context['object'].img_url:
            context['object'].img_url = kage_util.get_kage_full_url(self.request, context['object'].img_url)

        return context


class WeeklyBannerCreateView(PermissionMixin, CreateView):
    permission_required = permissions.ManageBanner()
    template_name = "hani/banner/weekly_create.html"
    model = TV4Banner
    fields = ['title', 'bgcolor', 'destination_type']
    url_params = dict()  # position, section_uid
    request = None
    errors = []

    def get(self, request, *args, **kwargs):
        self.request = request
        self.url_params = kwargs
        req_data = app_util.collect_request_data(self.request, ['POST', 'GET'], ['day'])
        if 'day' in req_data and req_data['day']:
            return super(WeeklyBannerCreateView, self).get(request, *args, **kwargs)
        else:
            return HttpResponseRedirect(reverse('weekly_banner_list', kwargs=kwargs))

    def post(self, request, *args, **kwargs):
        self.errors = []

        if 'img_data' in request.FILES and request.FILES['img_data']:
            section = TV4Section.objects.get(pk=kwargs['section_uid'])
            img_size_key = get_image_size_key(kwargs['position'], section.section_type)
            if is_valid_image(request.FILES['img_data'], self.errors, img_size_key):
                request.POST['img_url'] = kage_util.upload_to_kage(request.FILES['img_data'])

        if len(self.errors) == 0:
            with transaction.atomic():

                # Banner 생성
                banner = TV4Banner()
                banner.banner_type = 'BA01'  # 빅 배너
                banner_essentials = ["title", "destination_type", "destination", "img_url"]
                made_banner = app_util.make_model_obj(banner, request.POST, banner_essentials, ["bgcolor"])

                if made_banner['success']:
                    banner.save()
                    kwargs['pk'] = banner.uid

                    # Section item 자동생성
                    section = TV4Section.objects.get(uid=kwargs['section_uid'])
                    section_item_info = {'section': section, 'object_id': banner.uid, 'content_type_id': 10}

                    if 'is_always' in request.POST:  # 상시 배너 처리 (7개 스케줄 생성)
                        # day=0 이 아닌 모든 요일 배너의 item order 를 1씩 증가시킨다
                        TV4SectionItem.objects.filter(section=section, content_type=10).exclude(day=0).update(item_order=F('item_order') + 1)

                        for day_val in range(1, 8):
                            section_item_info['day'] = day_val
                            init_section_item = TV4SectionItem(**section_item_info)
                            init_section_item.save()

                    else:  # 처음 접근한 1개 요일 스케줄만 생성
                        # 해당요일의 item order 를 1씩 증가시킨다
                        TV4SectionItem.objects.filter(section=section, content_type=10, day=request.POST['day']).update(item_order=F('item_order') + 1)
                        section_item_info['day'] = request.POST['day']
                        init_section_item = TV4SectionItem(**section_item_info)
                        init_section_item.save()
                else:
                    self.errors.append({'msg': made_banner['message']})

        if len(self.errors) == 0:
            return HttpResponseRedirect(reverse('weekly_banner_detail', kwargs=kwargs) + "?day=" + request.POST['day'])
        else:
            return self.get(request, *args, **kwargs)

    def get_context_data(self, **kwargs):
        context = super(WeeklyBannerCreateView, self).get_context_data(**kwargs)
        context['errors'] = self.errors
        context['params'] = self.url_params
        req_data = app_util.collect_request_data(self.request, ['POST', 'GET'], ['day'])
        if 'day' in req_data:
            context['params']['day'] = req_data['day']
        section = TV4Section.objects.get(pk=self.url_params['section_uid'])
        context['img_size'] = BANNER_IMG_SIZE[get_image_size_key(self.url_params['position'], section.section_type)]
        context['os_choices'] = TV4SectionItem.OS_TYPES
        context['activation_choices'] = TV4SectionItem.ACTIVATIONS
        context['category_list'] = TCategory.objects.filter(parent_uid=0).exclude(uid=0).all()

        if 'object' not in context:
            context['object'] = {'uid': 0}
        if hasattr(self.request, 'POST'):
            recover_fields = ['destination_type', 'destination']
            app_util.add_posted_data_to_context(context, self.request.POST, recover_fields)

        return context
* 파이프 라인 방식 (html 내 오브젝트(이미지, 영상 등)들은 파이프라인 방식을 이용(한꺼번에 처리))
* Connection은 아래의 다섯가지 요인으로 식별, 이중 remote ip와 remote port를 제외한 나머지는 고정값

* 304 Not Modified는 서버에 요청할 때 반환될 값이 특정 date 이후로 변했는지를 체크해서 변한게 없다면 실제 데이터를 전송하지 않고 304 status code를 반환한다.

## 3강

* IP + PORT(프로세싱)
* DNS : host name과 IP 어드레스의 매핑 디렉토리, 전세계 곳곳에 분할되어 있음
* 성능상의 이유, 무수히 많은 데이터로 인해 분산되어 있고 계층화되어있다. (com DNS servers, org DNS servers … )
* DNS record : name(Host), value(IP), type, TTL(Time To Leave: 유효기간)
* type = A : 이름이 호스트네임, value가 IP address
* type = NS : name is domain, value는 authoritative name server의 호스트네임
* authoritative name server는 모두 Atype (ex. www.ha.edu, cse.ha.edu)
* dns.xxx.com 이라는 authoritative name server를 먼저 구성해야함 (아마 호스트 업체에서 대신 운영하는듯), 그리고 .com 네임서버에 등록 필요

4강

* 트랜스포터 레이어(TCP/UDP)
* Connectionless demux는 dest IP, port로 소켓을 분리, Connection demux(TCP)는 destIP, port, sourceIP, port 이렇게 4가지가 동일할때만 동일한 소켓을 열어줌.
* 멀티플렉싱은 하나의 전송로를 여러 사용자(클라이언트)가 동시에 사용해서 효율성을 높이는 것
* 여기서는 어플리케이션 레이어에서 특정 소켓을 찾아가는 과정을 뜻함, 반대로 디멀티플렉싱은 리시버에서 소켓에서 실제 프로세스를 찾아가는 과정?
* 실제로 구글에 접속할때 하나의 사용자에 하나의 소켓이 열리는게 아니라, 여러개의 쓰레드로 이뤄진 프로세스에 하나의 쓰레드와 연결됨
* DNS는 UDP 프로토콜을 사용함 —> why??
* UDP checksum은 데이터의 에러를 판단하기 위한 장치
* UDP에서 source port가 필요한 이유는 서버에서 return 해줄 주소를 알고 있어야 하므로
